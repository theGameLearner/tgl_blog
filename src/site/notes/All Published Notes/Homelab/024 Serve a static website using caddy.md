---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/024-serve-a-static-website-using-caddy/"}
---

created: 2026-08-04
updated: 2026-08-04

I currently have one extra caddy alpine image created in [[All Published Notes/Homelab/019 Static Page Display\|019 Static Page Display]] which only serves a single file.
I also have another caddy image-based container(`forward_proxy_caddy`) that can be used to mount the volumes as needed. The idea is that we use same caddy alpine image for both.

As this is an upgrade, I am assuming we have covered the volume mounting(`/apps/url_map`) in yml, if not re-visit the page: [[All Published Notes/Homelab/019 Static Page Display\|019 Static Page Display]].

I was also able to free up my port `10005` by moving the mapper file to the CoreDNS caddy file and using a command to use file_server to host the page directly on the existing caddy image.
To achieve this, 
- we defined a new `serve_static_site` importable snippet in `/apps/localDns/caddy/config/Caddyfile` and added a url mapping for the name we want to use to see this hosted file.
- In `/apps/localDns/coredns/Corefile`, we added a new line so that the URL will be managed by the IP address which has our caddy file.
### Steps
Stop or kill the container which we created in [[All Published Notes/Homelab/019 Static Page Display\|019 Static Page Display]], so we can use same URL or host same file without errors.

#### In yml file of CoreDNS

We will now use the same directory(`/apps/url_map`) as a mount point in existing caddy file's yml code:
```yml
    volumes:
      # Mapped to match your actual system configuration path structure
      - /apps/localDns/caddy/config/Caddyfile:/etc/caddy/Caddyfile:ro
      - /appdata/localDns/caddy/data2:/data
      # - /apps/localDns/caddy/data:/data # or /appdata/localDns/caddy/data2 if you encounter permission issue
      - /apps/localDns/caddy/config:/config
      - /appdata/localDns/certs:/certs:ro
      - /apps/url_map:/srv/url_map:ro
```

#### In Caddyfile
add a new code snippet on top:
```sh
   101  # =====================================================================         
   102  # Serve Files
   103  # =====================================================================         
   104  (serve_static_site) {
   105      # Load your Let's Encrypt Wildcard Certificates
   106      tls /certs/tglservice.top_ecc/fullchain.cer /certs/tglservice.top_ecc/tglservice.top.key
   107
   108      # Standardize access logs to stdout
   109      log {
   110          output stdout
   111      }
   112
   113      # Set the folder path passed as {args[0]} and enable static serving
   114      root * {args[0]}
   115      file_server
   116  }
   117
```

Add the line which serves the file_server on a url:
```sh
   158  # 7. Server File Map
   159  map.tglservice.top map.nb.tglservice.top {
   160      import serve_static_site /srv/url_map
   161  }
```

#### In Corefile
Add the entry which tells the system which DNS will handle the URL request.
```sh
    14          192.168.1.100 map.tglservice.top
    15          192.168.1.100 diffchecker.tglservice.top
```

restart the coreDNS stack.
#### Content
Instead of serving a running server, we are using a file_server to return a index.html file which will then show the content of `url_mapping` file: 
```sh
Tue Aug 04, 04:52:17 PM "~"|root@DockerHost:# ls -al /apps/url_map
total 512
drwxrwxrwx 2 nobody nogroup 131072 Jul 11 14:03 .
drwxrwxrwx 7 nobody nogroup 131072 Aug  1 15:02 ..
-rwxrwxrwx 1 nobody nogroup    559 Jul 11 14:52 index.html
-rwxrwxrwx 1 nobody nogroup   1281 Aug  4 16:52 url_mapping.md

Tue Aug 04, 05:15:19 PM "~"|root@DockerHost:# ls -al /apps/url_map/index.html 
-rwxrwxrwx 1 nobody nogroup 559 Jul 11 14:52 /apps/url_map/index.html

Tue Aug 04, 05:15:33 PM "~"|root@DockerHost:# cat -n /apps/url_map/index.html 
     1  <!DOCTYPE html>
     2  <html>
     3  <head>
     4    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
     5    <meta name="viewport" content="width=device-width,initial-scale=1">
     6    <meta charset="UTF-8">
     7    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/docsify@4/themes/dark.css">
     8    <title>Server Mapping</title>
     9  </head>
    10  <body>
    11    <div id="app"></div>
    12    <script>
    13      window.$docsify = {
    14        homepage: 'url_mapping.md' // Tells Docsify to load the specific file
    15      }
    16    </script>
    17    <script src="https://cdn.jsdelivr.net/npm/docsify@4"></script>
    18  </body>
    19  </html>
    20

Tue Aug 04, 05:15:41 PM "~"|root@DockerHost:# 
```

### Summary
This file is hosted in Caddyfile and published to us based on the IP address sent. So, we do not use an IP and port combination, rather the file is delivered to us in standard HTTP/HTTPS ports (80/443), and Caddy immediately hands back the file from RAM/disk without doing any internal network round-trips to local ports.




---

[^1]: [[All Published Notes/Homelab/023a diff checker offline - monaco-editor\|023a diff checker offline - monaco-editor]]
[^2]: 

