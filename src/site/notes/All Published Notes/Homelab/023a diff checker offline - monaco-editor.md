---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/023a-diff-checker-offline-monaco-editor/"}
---

created: 2026-08-04
updated: 2026-08-04

> [!Warning]
> Though the file failed to work, I am storing it as a reference.
> If you want something like tutorial for how to achieve the same, look for other files covering the same topic like [[All Published Notes/Homelab/023b diff checker offline - text-diff-view\|text-diff]].

### Steps
#### Download the files
We need `monaco-editor` files locally, if we want to make the system offline. 
We will store all files in `/apps/diff_check/` directory, so that I have an exact address of the files.
```sh
# 1. Create folders
mkdir -p /apps/diff_check/vs/base/worker

# 2. Download Monaco loader & core scripts
wget -O /apps/diff_check/vs/loader.min.js https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.45.0/min/vs/loader.min.js
wget -O /apps/diff_check/vs/editor.main.js https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.45.0/min/vs/editor/editor.main.js
wget -O /apps/diff_check/vs/editor.main.css https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.45.0/min/vs/editor/editor.main.css
wget -O /apps/diff_check/vs/editor.main.nls.js https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.45.0/min/vs/editor/editor.main.nls.js
wget -O /apps/diff_check/vs/base/worker/workerMain.js https://cdnjs.cloudflare.com/ajax/libs/monaco-editor/0.45.0/min/vs/base/worker/workerMain.js
```

#### Create index file
Create index.html file:
```sh
Sat Aug 01, 03:02:55 PM "~"|root@DockerHost:# fresh /apps/diff_check/index.html

A new version of fresh is available: 0.3.5 -> 0.4.6
Download from: https://github.com/sinelaw/fresh/releases/tag/v0.4.6


Sat Aug 01, 03:05:47 PM "~"|root@DockerHost:# 
```

the file:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Offline Monaco Diff Checker</title>
  <style>
    html, body { margin: 0; padding: 0; height: 100%; width: 100%; overflow: hidden; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }
    #header { height: 42px; background: #1e1e1e; color: #cccccc; display: flex; align-items: center; padding: 0 16px; justify-content: space-between; border-bottom: 1px solid #333333; box-sizing: border-box; }
    #container { height: calc(100vh - 42px); width: 100vw; }
  </style>
  <script src="./vs/loader.min.js"></script>
</head>
<body>
  <div id="header">
    <strong>Homelab Diff Checker</strong>
    <span style="font-size: 12px; color: #888888;">100% Offline • Monaco Engine</span>
  </div>
  <div id="container"></div>

  <script>
    require.config({ paths: { 'vs': './vs' } });

    require(['vs/editor/editor.main'], function () {
      const diffEditor = monaco.editor.createDiffEditor(document.getElementById('container'), {
        theme: 'vs-dark',
        automaticLayout: true,
        renderSideBySide: true,
        originalEditable: true,
        useInlineViewWhenSpaceIsLimited: false
      });

      diffEditor.setModel({
        original: monaco.editor.createModel('// Paste original text here...', 'plaintext'),
        modified: monaco.editor.createModel('// Paste modified text here...', 'plaintext')
      });
    });
  </script>
</body>
</html>
```

#### Deploy
##### Deploy with new caddy container
We can make a new caddy file which will host this web page with port as `1006`:
```yml
version: '3.8'

services:
  monaco_diff:
    image: caddy:alpine
    container_name: caddy_page_monaco_diff_checker
    restart: unless-stopped
    command: caddy file-server --listen :80 --root /srv
    ports:
      - "10006:80"
    volumes:
      - /apps/diff_check:/srv:ro
```
OR
```yml
version: "3.7"

services:
  diff-checker:
    image: caddy:alpine
    container_name: caddy_page_monaco_diff_checker
    environment:
      - TZ=Asia/Kolkata
    ports:
      - "10006:80"
    volumes:
      - /apps/diff_check:/usr/share/caddy:ro
    command: caddy file-server --listen :80 --root /usr/share/caddy
    restart: unless-stopped
```

Once done, we can update our `CoreDNS` and `Caddyfile` to map this IP and port to a url.

##### Deploy with existing caddy container
If we already have a caddy file running, like in my case I have caddy images running as forward proxy (with coreDNS), we can add the configuration inside the Caddyfile. The only change in stack is adding the volume where the data resides, if it is not already accessible.

Add `- /apps/diff_check:/srv/diff_check:ro` to the volumes section of `forward_proxy_caddy` container:
```yml
version: "3"

services:
  coredns:
    image: coredns/coredns:latest
    container_name: dns_coredns
    restart: unless-stopped
    ports:
      # Binds CoreDNS cleanly on port 53 ONLY to your physical LAN IP interface
      # This prevents any binding collisions with Netbird running on your mesh interface
      - "192.168.1.100:53:53/udp"
      - "192.168.1.100:53:53/tcp"
    volumes:
      # Maps your custom regex configuration directly into the CoreDNS container runtime path in a read-only mode(ro)
      - /apps/localDns/coredns/Corefile:/Corefile:ro
    command: -conf /Corefile

  acme_sh:
    image: neilpang/acme.sh:latest
    container_name: cert_updater_acme_caddy 
    restart: unless-stopped
    environment:
      # Exact case-sensitive environment flags required by acme.sh
      - SPACESHIP_API_KEY=Pfdqb1gXPTuiIJjU6mQz
      - SPACESHIP_API_SECRET=2WdMNmJRTZMTnj1u9HOF9yyY6oK83mKpATI7PS80zYVKR0fm7RpBdok8Jl542XaQ
    volumes:
      - /appdata/localDns/certs:/acme.sh
    # Added --keylength ec-256 to ensure it builds an ECC cert matching the Caddy paths
    entrypoint: /bin/sh -c "acme.sh --upgrade --auto-upgrade && acme.sh --issue --dns dns_spaceship -d tglservice.top -d '*.tglservice.top' -d '*.nb.tglservice.top' --keylength ec-256 --server letsencrypt; crond -f"

  caddy:
    image: caddy:2.7-alpine
    container_name: forward_proxy_caddy
    restart: unless-stopped
    ports:
      # Listens on Port 80 across all interfaces (both LAN and Netbird)
      # This allows Caddy to catch incoming traffic no matter which network path is used
      - "80:80" # http
      - "443:443" # https
    volumes:
      # Mapped to match your actual system configuration path structure
      - /apps/localDns/caddy/config/Caddyfile:/etc/caddy/Caddyfile:ro
      - /appdata/localDns/caddy/data2:/data
      # - /apps/localDns/caddy/data:/data # or /appdata/localDns/caddy/data2 if you encounter permission issue
      - /apps/localDns/caddy/config:/config
      - /appdata/localDns/certs:/certs:ro
      - /apps/diff_check:/srv/diff_check:ro
      
```

Notice, we added `/srv/diff_check` so, in the caddyfile(`/apps/localDns/caddy/config/Caddyfile`) we need to add the same with the URL(`diff.tglservice.top`):
```
# 8. Diff Checker (Served directly from main proxy)
diff.tglservice.top, diff.nb.tglservice.top {
    tls /certs/tglservice.top_ecc/fullchain.cer /certs/tglservice.top_ecc/tglservice.top.key
    log {
        output stdout
    }
    root * /srv/diff_check
    file_server
}
```

Finally, in corefile, we can add a line telling that our IP address will handle the URL `diff.tglservice.top`:
```
192.168.1.100 diff.tglservice.top
```


##### Improvements
###### Caddyfile
Make a static file snippet in Caddyfile for serving static pages. This helps us avoid re-writing the same commands every time a new file needs to be added:
```
(serve_static_site) {
    # Load your Let's Encrypt Wildcard Certificates
    tls /certs/tglservice.top_ecc/fullchain.cer /certs/tglservice.top_ecc/tglservice.top.key

    # Standardize access logs to stdout
    log {
        output stdout
    }

    # Set the folder path passed as {args[0]} and enable static serving
    root * {args[0]}
    file_server
}
```

Here `args[0]` can be `/srv/diff_check` but if we have another site, we can just change the argument and the site will be served without any issues.

#### Corrections

```sh
# 1. Create the expected subdirectories
mkdir -p /apps/diff_check/vs/editor
mkdir -p /apps/diff_check/vs/base/worker

# 2. Move the editor main files into vs/editor
mv /apps/diff_check/vs/editor.main* /apps/diff_check/vs/editor/

# 3. copy the files so base path assets load without issue
cp -r /apps/diff_check/vs/editor/* /apps/diff_check/vs/base/worker/
```


### Result
I have a blank page, I attempted to make it work, but it is failing.

#### Cleanup
Delete the files downloaded in `/apps/diff_check` directory.
```
rm -rf /apps/diff_check/*
```

In `/apps/localDns/caddy/config/Caddyfile`, remove the following lines:
```sh
   163  # 8. Serve File diff checker
   164  diffchecker.tglservice.top, diffchecker.nb.tglservice.top {
   165      import serve_static_site /srv/diff_check
   166  }
```

In `/apps/localDns/coredns/Corefile`, remove the line about the new service:
```sh
    15          192.168.1.100 diffchecker.tglservice.top
```

Finally, remove the volume in `forward_proxy_caddy` container in the yml file of `local-dns` stack.





---

[^1]: 
[^2]: 

