---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/017-using-filestash-to-access-samba-share/"}
---

created: 2026-07-05
updated: 2026-07-05

The problem I had with 'file browser'[^1] was that there was no guest, I had to manually control the access for each directory, so even if I was able to create a simple guest with access to my directories, I cannot give access to 2 directories without doing a re-architecture of how my drives are located.

I have since found a solution called 'Filestash' which will act as a login interface for users to access my Samba files and I can define the controls in Samba. 

As the system is created after I have tried to create 'file browser'[^1], you can check [[All Published Notes/Homelab/016 Using filebrowser instead of samba#Understanding Current state *(Optional)*\|Current state]] from that file.

Steps:
- [[#Prepare the directory structure on LXC]]
- [[#Find the Samba share URL]]
- [[#Create yml for Filestash]]
- [[#In filestash, add admin credentials]]
- [[#add samba as backend for Filestash]]
- [[#Testing]]

## Prepare the directory structure on LXC

Create a directory which will act as the application folder for FileStash:
```sh
root@DockerHost:~# ls -al /apps/
total 388
drwxrwxrwx  4 nobody nogroup 131072 Jul  4 14:57 .
drwxr-xr-x 24 root   root      4096 Jul  4 14:25 ..
drwxrwxrwx  2 nobody nogroup 131072 Jul  4 15:34 file_browser
drwxrwxrwx  4 nobody nogroup 131072 Jun 19 19:01 localDns
root@DockerHost:~# mkdir -p /apps/filestash/data
root@DockerHost:~# mkdir -p /apps/filestash/config
root@DockerHost:~# ls -al /apps/
total 516
drwxrwxrwx  5 nobody nogroup 131072 Jul  4 19:45 .
drwxr-xr-x 24 root   root      4096 Jul  4 14:25 ..
drwxrwxrwx  2 nobody nogroup 131072 Jul  4 15:34 file_browser
drwxrwxrwx  4 nobody nogroup 131072 Jul  4 19:45 filestash
drwxrwxrwx  4 nobody nogroup 131072 Jun 19 19:01 localDns
root@DockerHost:~# 
```

## Find the Samba share URL
To find the URL, first let us identify the IP address of all devices:
```sh
root@pve:~# hostname -I
192.168.1.8 192.168.1.101 100.77.214.80 2401:4900:1cb8:a05f:de56:7bff:fed3:def 
root@pve:~# pct enter 400
root@NAS-LXC:~# hostname -I
192.168.1.200 100.77.223.52 2401:4900:1cb8:a05f:be24:11ff:fed6:4b8b 
root@NAS-LXC:~# exit
exit
root@pve:~# pct enter 401
root@DockerHost:~# hostname -I
192.168.1.100 100.77.8.142 172.19.0.1 172.20.0.1 172.18.0.1 172.17.0.1 172.21.0.1 
root@DockerHost:~# 
```
So, LXC 400 with Samba configured in it uses IP '192.168.1.200', so samba share has url: `smb://192.168.1.200`


## Create yml for Filestash
Create docker yml
Old one, had issues later
```yml
version: '3.8'

services:
  filestash: # service header
    image: machines/filestash:latest # image to use
    container_name: filestash # name as seen in docker ps
    ports:
      - "10003:8334" # using port 10003
    environment:
      - APPLICATION_URL=http://192.168.1.100:10003 # IP address + port of current LXC
    volumes:
      - /apps/filestash/data:/app/data # /apps/filestash/data in LXC for data
      - /apps/filestash/config:/app/config # /apps/filestash/config in LXC for config
    restart: unless-stopped
```

new one:
```sh
version: '3.8'

services:
  filestash: # service header
    image: machines/filestash:latest # image to use
    container_name: filestash # name as seen in docker ps
    ports:
      - "10003:8334" # using port 10003
    environment:
      - APPLICATION_URL= "" # http://192.168.1.100:10003 # IP address + port of current LXC
    volumes:
      - /apps/filestash/data:/app/data/state
    restart: unless-stopped
```

## In filestash, add admin credentials

Now, the fileStash is active on `http://192.168.1.100:10003`
![063 homelab filestash admin pasword.png](/img/user/All%20Published%20Notes/Homelab/Images/063%20homelab%20filestash%20admin%20pasword.png)
Setup an admin password and move to console, we are using http, but we can ignore the warning for now
![064 homelab filestash dashboard.png](/img/user/All%20Published%20Notes/Homelab/Images/064%20homelab%20filestash%20dashboard.png)

## add samba as backend for Filestash

Now, that the filestash container is up and running, we want to use our local NAS IP (192.168.1.200) to access the samba share.

![065 homelab filestash attaching data source.png](/img/user/All%20Published%20Notes/Homelab/Images/065%20homelab%20filestash%20attaching%20data%20source.png)

Add 3 labels for the 3 drives to connect to:

![066 homelab filestash adding label.png](/img/user/All%20Published%20Notes/Homelab/Images/066%20homelab%20filestash%20adding%20label.png)

Add backend for each drive:
![067 homelab filestash adding mapping.png](/img/user/All%20Published%20Notes/Homelab/Images/067%20homelab%20filestash%20adding%20mapping.png)

We used `{{ .user }}` and `{{ .password }}` so that whatever the user types gets passed to the login portal of Samba as well.

## Testing

Open the URL `http://192.168.1.100:10003/` and you should be re-directed to login page:

![068 homelab filestash testing 01.png](/img/user/All%20Published%20Notes/Homelab/Images/068%20homelab%20filestash%20testing%2001.png)

Select the 'Games' label
![069 homelab filestash testing 02.png](/img/user/All%20Published%20Notes/Homelab/Images/069%20homelab%20filestash%20testing%2002.png)

Click connect without entering any username or password:
![070 homelab filestash testing 03.png](/img/user/All%20Published%20Notes/Homelab/Images/070%20homelab%20filestash%20testing%2003.png)

This allows to access media and games without any credentials, just like in samba. We can close a connection with power button on top right:
![071 homelab filestash testing 04.png](/img/user/All%20Published%20Notes/Homelab/Images/071%20homelab%20filestash%20testing%2004.png)

but trying to connect to backup without credentials shows a empty file:
![072 homelab filestash testing 05.png](/img/user/All%20Published%20Notes/Homelab/Images/072%20homelab%20filestash%20testing%2005.png)

Final attempt, use the account username and password for samba user to login to the drive from filestash.
It worked!!

So, now I have a service that allows me to access the drives using an IP address!

## Making a URL for FileStash

As the system now works from a URL, we can add it to our 'caddy' container for access.
open the caddy file (`/apps/localDns/caddy/config/Caddyfile`) and add file stash as a service:
```sh
root@DockerHost:~# cat -n /apps/localDns/caddy/config/Caddyfile
     1  # =====================================================================
     2  # GLOBAL OPTIONS BLOCK
     3  # =====================================================================
     4  {
     5      # Stops Caddy from trying to request public Let's Encrypt certs for private IPs
     6      local_certs
     7  }
     8
...
    43
    44  # ===================================================================== 
    45  # DEFINE YOUR SNIPPET (ONCE AT THE VERY TOP)
    46  # defines the certification path local to caddy container
    47  # defines the common setup needed for a url
    48  # {args.0} = Main LAN IP & Port (e.g., 192.168.1.100:9443)
    49  # {args.1} = Netbird Mesh IP & Port (e.g., 100.77.8.142:9443)
    50  # ===================================================================== 
    51
    52  (secure_proxy_https) {
...
    76  }
    77
    78  (secure_proxy_http) {
...
    95  }
    96
    97
    98  # =====================================================================
    99  # REVERSE PROXY & HOST DEFINITIONS
   100  # Note: 
   101  #    - Since Vikunja uses standard HTTP upstream, we append "http://" 
   102  #      to the args to tell Caddy not to use HTTPS on the back-end connection.
   103  #    - Browser uses secure connection so we use "https://"
   104  # =====================================================================
   105
   106  # 1. Vikunja Task Management Service  
   107  vikunja.tglservice.top, vikunja.nb.tglservice.top {
   108      import secure_proxy_http http://192.168.1.100:3456 http://100.77.8.142:3456
   109  }
   110           
   111  # 2. Portainer Management Dashboard
   112  portainer.tglservice.top, portainer.nb.tglservice.top {         
   113      import secure_proxy_https https://192.168.1.100:9443 https://100.77.8.142:9443
   114  }        
   115           
   116  # 3. Isolated Dedicated Browser Container
   117  browser.tglservice.top, browser.nb.tglservice.top {                 
   118      import secure_proxy_https https://192.168.1.100:10001 https://100.77.8.142:10001
   119  }  
   120
   121  # 4. Proxmox interface
   122  proxmox.tglservice.top proxmox.nb.tglservice.top {
   123      import secure_proxy_https https://192.168.1.101:8006 https://100.77.214.80:8006
   124  }
   125
   126  # 5. FileStash interface
   127  nas.tglservice.top nas.nb.tglservice.top {
   128      import secure_proxy_http http://192.168.1.100:10003 https://100.77.8.142:10003
   129  }
   130   
   131  # ===================================================================== 
   132  # UTILITY AND FALLBACK BLOCKS
   133  # =====================================================================
   134  # n-1. Global Health Checker
   135  health.tglservice.top, health.nb.tglservice.top {   
   136      tls /certs/tglservice.top_ecc/fullchain.cer /certs/tglservice.top_ecc/tglservice.top.key
   137      log { 
   138          output stdout 
   139      } 
   140      respond "OK" 200      
   141  }        
   142
   143  # n. Catch-All Default 404 Error Block
   144  *.tglservice.top, tglservice.top, *.nb.tglservice.top, nb.tglservice.top {
   145      tls /certs/tglservice.top_ecc/fullchain.cer /certs/tglservice.top_ecc/tglservice.top.key
   146      log {
   147          output stdout
   148      }
   149      respond "Service Not Found" 404 
   150  #    {
   151  #        close    
   152  #    }        
   153  }
   154
root@DockerHost:~#
```

restart caddy and test:
```sh
root@DockerHost:~# fresh /apps/localDns/caddy/config/Caddyfile
root@DockerHost:~# docker restart forward_proxy_caddy
forward_proxy_caddy
root@DockerHost:~# docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED             STATUS                 PORTS                                                                                                NAMES
e2adf0b10039   machines/filestash:latest          "/app/filestash"         About an hour ago   Up About an hour       0.0.0.0:10003->8334/tcp, [::]:10003->8334/tcp                                                        filestash
fb21b2fa1456   filebrowser/filebrowser:latest     "tini -- /init.sh"       4 hours ago         Up 4 hours (healthy)   0.0.0.0:10002->80/tcp, [::]:10002->80/tcp                                                            filebrowser
6d43412eb1b4   neilpang/acme.sh:latest            "/bin/sh -c 'acme.sh…"   11 days ago         Up 8 hours                                                                                                                  cert_updater_acme_caddy
93ce91ee0800   caddy:2.7-alpine                   "caddy run --config …"   13 days ago         Up 1 second            0.0.0.0:80->80/tcp, [::]:80->80/tcp, 0.0.0.0:443->443/tcp, [::]:443->443/tcp, 443/udp, 2019/tcp      forward_proxy_caddy
283a47a25714   coredns/coredns:latest             "/coredns -conf /Cor…"   13 days ago         Up 8 hours             192.168.1.100:53->53/tcp, 192.168.1.100:53->53/udp                                                   dns_coredns
22359eb3e46c   lscr.io/linuxserver/brave:latest   "/init"                  6 weeks ago         Up 8 hours             0.0.0.0:10000->3000/tcp, [::]:10000->3000/tcp, 0.0.0.0:10001->3001/tcp, [::]:10001->3001/tcp         risky_browser
36e6b22d3fa3   vikunja/vikunja                    "/app/vikunja/vikunja"   7 weeks ago         Up 8 hours             0.0.0.0:3456->3456/tcp, [::]:3456->3456/tcp                                                          vikunja-vikunja-1
169f8a926fbc   postgres:16-alpine                 "docker-entrypoint.s…"   7 weeks ago         Up 8 hours             5432/tcp                                                                                             vikunja-db-1
0ddedadb09f8   portainer/portainer-ce:latest      "/portainer"             7 weeks ago         Up 8 hours             0.0.0.0:8000->8000/tcp, [::]:8000->8000/tcp, 0.0.0.0:9443->9443/tcp, [::]:9443->9443/tcp, 9000/tcp   portainer_container
root@DockerHost:~# 
```

Now, we can access it over LAN on any machine, and I only need to use password when I want to write something new.

> [!Note]
> Trying the same with https failed, I am not sure of the reason and do not want to worry about it given that the whole system is running locally with no external access.


---

[^1]: [[All Published Notes/Homelab/016 Using filebrowser instead of samba\|016 Using filebrowser instead of samba]]
[^2]: 

