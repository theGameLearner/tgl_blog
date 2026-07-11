---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/019-static-page-display/"}
---

created: 2026-07-11
updated: 2026-07-11

With so many services being added, I need 'one' page with all service URL and the IP addresses that are hosted on my homelab.

As the connection is secure, we will also add if there is a public password, as in any password which I do not need but is part of the setup and hence unavoidable, for e.g., qBittorrent.

The tool I am using for this is [docsify](https://docsify.js.org) which will allow me to host my page as a standalone markdown(`.md`) document and can see it on a page.

### Create the server folder

The app will serve a directory as a site, so I will create that:
```sh
Sat Jul 11, 01:56:40 | root@DockerHost:"~"# mkdir -p /apps/url_map
```
First I will create a `.md` file which I want to see in a static page:
```sh
Sat Jul 11, 01:57:55 | root@DockerHost:"~"# ls -al /apps/url_map/
total 256
drwxrwxrwx 2 nobody nogroup 131072 Jul 11 13:57 .
drwxrwxrwx 6 nobody nogroup 131072 Jul 11 13:57 ..
-rwxrwxrwx 1 nobody nogroup      0 Jul 11 13:57 url_mapping.md
```

Now I need a `index.html` page here for defining the web site look and feel:
```sh
Sat Jul 11, 02:26:24 | root@DockerHost:"~"# cat -n /apps/url_map/index.html
     1  <!DOCTYPE html>
     2  <html>
     3  <head>
     4    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
     5    <meta name="viewport" content="width=device-width,initial-scale=1">
     6    <meta charset="UTF-8">
     7    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/docsify@4/themes/vue.css">
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
Sat Jul 11, 02:26:32 | root@DockerHost:"~"# 
```

We are using the script from [jsdelivr](cdn.jsdelivr.net/npm/docsify@4/) to use '[vue.css](https://cdn.jsdelivr.net/npm/docsify@4/themes/vue.css)' as the theme, you can change the theme if you wish.

### Create the container to server the server

Given that a server is hosting data on a site and we already use caddy, I will serve the mapper as a caddy container serving the file that we created.

the yml(docker compose or portainer stack file):
```yml
version: "3.7"

services:
  docsify-dashboard:
    image: caddy:alpine
    container_name: docsify_mapper
    environment:
      - TZ=Asia/Kolkata
    ports:
      - "10005:80"
    volumes:
      - /apps/url_map:/usr/share/caddy
    command: caddy file-server --listen :80 --root /usr/share/caddy
    restart: unless-stopped
```

Now if we open our local IP address with port `10005`, we should see the content of `url_mapping.md` on the webpage.

Next, we need to define the URL for the IP address so we update the caddy and corefile:
- /apps/localDns/caddy/config/Caddyfile
- /apps/localDns/coredns/Corefile




---

[^1]: 
[^2]: 

