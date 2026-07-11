---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/018-installing-qbittorrent-for-torrents/"}
---

created: 2026-07-10
updated: 2026-07-11

As every developer and movie lover knows, downloading files on torrent is a common hobby of users. It made us feel like hackers as a kid and feels essential even today.

I will try to install qBittorrent on docker so I can put files in my hard drives using torrent, and given that I have access with samba and filestash, all downloaded files can be opened easily.

The docker compose file need to know where to save the downloaded data:
```sh
Fri Jul 10, 03:30:18 | root@DockerHost:"~"# ls -al /mnt/
total 392
drwxr-xr-x  5 root   root      4096 Jul  4 14:25 .
drwxr-xr-x 24 root   root      4096 Jul  4 14:25 ..
drwxrwxrwx  9 nobody nogroup 131072 Jun 28 08:03 hdd-games
drwxrwxrwx 18 nobody nogroup 131072 Jul  4 19:49 hdd-main-backup
drwxrwxrwx  9 nobody nogroup 131072 Jul  4 19:49 hdd-media
```

The docker compose or portainer stack file:
```yml
version: "2.1" # docker yml version
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent # name of the service
    environment:
      - PUID=2300          # Matches 'dockeruser' user id
      - PGID=990           # Matches 'docker' group id
      - TZ=Asia/Kolkata    # India timezone
      - WEBUI_PORT=10004   # Tells qBittorrent internal webUI to run on 10004
    volumes:
      - /home/dockeruser/qbittorrent/config:/config
      # Mount the parent /mnt directory so the container sees all 3 HDDs at once
      - /mnt:/downloads    
    ports:
      - 10004:10004        # WebUI routing
      - 6881:6881          # Torrent traffic (TCP)
      - 6881:6881/udp      # Torrent traffic (UDP)
    restart: unless-stopped
```

now, when we start it, the system generates a password for admin account(`docker logs qbittorrent`), which I do not like entering every time.

So, now we will add the following lines to change the setting so that anyone accessing it on local IP should be considered authorized. 

> [!Warning]
> Do this only if you have no security issues, as my Wi-Fi is physically secure and I only access data from Netbird VPN, I believe the security risk is minimized here.

Steps:
- stop any running container, as container overwrites the config during shutdown
- Change the config data to whitelist all users in 192.168 mask
- start the container again

Change the file to update the config file: 

```sh
Fri Jul 10, 05:32:14 | root@DockerHost:"~"# docker stop qbittorrent
qbittorrent
Fri Jul 10, 05:34:29 | root@DockerHost:"~"# fresh /home/dockeruser/qbittorrent/config/qBittorrent/qBittorrent.conf 
Fri Jul 10, 05:34:45 | root@DockerHost:"~"# cat -n /home/dockeruser/qbittorrent/config/qBittorrent/qBittorrent.conf 
     1  [AutoRun]
     2  enabled=false
     3  program=
     4
     5  [BitTorrent]
     6  Session\DefaultSavePath=/downloads/
     7  Session\Port=6881
     8  Session\QueueingSystemEnabled=true
     9  Session\SSL\Port=4941
    10  Session\TempPath=/downloads/incomplete/
    11
    12  [LegalNotice]
    13  Accepted=true
    14
    15  [Meta]
    16  MigrationVersion=8
    17
    18  [Network]
    19  Cookies=@Invalid()
    20  PortForwardingEnabled=false
    21  Proxy\HostnameLookupEnabled=false
    22  Proxy\Profiles\BitTorrent=true
    23  Proxy\Profiles\Misc=true
    24  Proxy\Profiles\RSS=true
    25
    26  [Preferences]
    27  Connection\PortRangeMin=6881
    28  Connection\UPnP=false
    29  Downloads\SavePath=/downloads/
    30  Downloads\TempPath=/downloads/incomplete/
    31  WebUI\Address=*
    32  WebUI\Port=10004
    33  WebUI\ServerDomains=*
    34  WebUI\Username=admin
    35  WebUI\AuthSubnetWhitelist=192.168.0.0/16, 10.0.0.0/8
    36  WebUI\AuthSubnetWhitelistEnabled=true
    37  WebUI\LocalHostAuth=false
    38
Fri Jul 10, 05:34:48 | root@DockerHost:"~"# docker start qbittorrent
qbittorrent
Fri Jul 10, 05:35:13 | root@DockerHost:"~"# docker logs qbittorrent
```

The password is still generated, but you only need it if you are not on local WiFi, so I do not need it at home, and I only need it if I am using VPN to access my homelab from outside.

Let's add it to caddy and coreDNS with `http` as the access as I have not used any secure setting.
- /apps/localDns/caddy/config/Caddyfile
- /apps/localDns/coredns/Corefile

After this is done, we can set the password:
![073 homelab qbittorrent options.png](/img/user/All%20Published%20Notes/Homelab/Images/073%20homelab%20qbittorrent%20options.png)

![074 homelab qbittorrent password.png](/img/user/All%20Published%20Notes/Homelab/Images/074%20homelab%20qbittorrent%20password.png)

Now we can set a password and use it unless the container is deleted and re-built.






---

[^1]: 
[^2]: 

