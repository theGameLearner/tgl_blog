---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/026-installing-ru-torrent/"}
---

created: 2026-08-15
updated: 2026-08-15

I have a torrent client running: [[All Published Notes/Homelab/018 Installing qbittorrent for torrents\|018 Installing qbittorrent for torrents]] but, the torrent does not show folder structure, and trying to type the exact directories with exact cases has become an issue.

I tried to find alternatives and decided to go with 'rutorrent'(`crazymax/rutorrent`).

As I have created a directory where all my hard drives are present:
```sh
Sat Aug 15, 04:32:07 AM "~"|root@DockerHost:# pwd
/root

Sat Aug 15, 04:32:12 AM "~"|root@DockerHost:# ls -al /mnt
total 148
drwxr-xr-x  6 dockeruser docker    4096 Aug 13 05:52 .
drwxr-xr-x 24 root       root      4096 Jul 26 08:56 ..
drwxrwxrwx  3 nobody     nogroup   4096 Jan  3  2026 hdd-games
drwxrwxrwx 18 nobody     nogroup 131072 Jul  4 19:49 hdd-main-backup
drwxrwxrwx 10 nobody     nogroup   4096 Jan  4  2026 hdd-media
drwxr-xr-x  3 dockeruser docker    4096 Aug 13 05:52 mnt

Sat Aug 15, 04:32:14 AM "~"|root@DockerHost:# 
```

And my torrent client is already running on `https://torrent.tglservice.top/` url for port 10004. I will update the existing stack to use rutorrent on same IP and port. As my URL does not specify a image name, and only mentions this is a torrent client, I should be able to use it without much change except for the yml file

Delete the old stack and container before creating the new files.

the new yml file:
```yml
services:
  rutorrent:
    image: crazymax/rtorrent-rutorrent:latest
    container_name: rutorrent
    environment:
      - PUID=2300
      - PGID=990
      - TZ=Asia/Kolkata
    volumes:
      - /home/dockeruser/rutorrent/config:/data
      - /mnt:/downloads
    ports:
      - 10004:8080    # ruTorrent WebUI
      - 6881:6881
      - 6881:6881/udp
    restart: unless-stopped
```

















---

[^1]: 
[^2]: 

