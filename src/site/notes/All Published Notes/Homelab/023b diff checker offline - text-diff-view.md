---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/023b-diff-checker-offline-text-diff-view/"}
---

created: 2026-08-04
updated: 2026-08-04

### Current Situation
After the failed attempt made earlier ([[All Published Notes/Homelab/023a diff checker offline - monaco-editor\|monaco-editor]]), I am switching attempt to use `text-diff-view` docker image from 'kaishuu0123'. 
- I currently have a directory defined in which I will store any data if needed : `/apps/diff_check/`
- I have coreDNS running, which can be used to re-direct a URL to a IP or port: `/apps/localDns/coredns/Corefile`
- I also have a well configured caddy file which we can use to define how the LXC handles IP and port mapping in my existing router: `/apps/localDns/caddy/config/Caddyfile` 
Demo by the creator: [link](https://sandbox.saino.me/text-diff-view/).

### Steps
#### Set up compose file (yml)
create a new docker compose file:
```yml
version: '3.8'

services:
  diff-check:
    image: ghcr.io/kaishuu0123/text-diff-view:latest
    container_name: diff_checker_text
    restart: unless-stopped
    ports:
      - "10005:80" # Maps host port 10005 to container port 80
```


#### Update the caddy file
Add entry to see this file in `/apps/localDns/caddy/config/Caddyfile`:
```sh
   163  # 8. Text Diff View Tool
   164      diffchecker.tglservice.top, diffchecker.nb.tglservice.top {
   165      import insecure_proxy_http http://192.168.1.100:10005 http://100.77.8.142:10005
   166  }
   167
```

#### Tell the system who manages DNS
In `/apps/localDns/coredns/Corefile` file, add entry for this URL:
```sh

Tue Aug 04, 04:48:01 PM "~"|root@DockerHost:# cat -n /apps/localDns/coredns/Corefile
     1  .:53 {
     2      log
     3      errors
     4
     5      # Use hosts file for exact tglservice names (no regex needed)
     6      hosts {
     7          192.168.1.100 portainer.tglservice.top
     8          192.168.1.100 vikunja.tglservice.top
     9          192.168.1.100 browser.tglservice.top
    10          192.168.1.100 health.tglservice.top
    11          192.168.1.100 proxmox.tglservice.top
    12          192.168.1.100 nas.tglservice.top
    13          192.168.1.100 torrent.tglservice.top
    14          192.168.1.100 map.tglservice.top
    15          192.168.1.100 diffchecker.tglservice.top
```

Remember to re-start your caddy file and core file stacks to test.
### Result
The new diff checker works flawlessly, though I do not know if it will work without internet.



---

[^1]: 
[^2]: 

