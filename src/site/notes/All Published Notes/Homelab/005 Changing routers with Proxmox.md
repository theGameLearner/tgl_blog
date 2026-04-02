---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/005-changing-routers-with-proxmox/"}
---

created: 2026-04-03
updated: 2026-04-03

I had this experience so sharing the same here.
I changed my ISP from Jio to Airtel, meaning a new router and new setup. After switching on my proxmox machine, I was unable to log into `https://192.168.31.101:8006` which was set earlier. The reason was that my new router uses the mask `192.168.1.x` instead of `192.168.31.x`.

The Solution: I had to change the configurations in my proxmox as `root` user in `/etc/hosts`(`192.168.1.101 pve.lan pve`) and `/etc/network/interfaces`('address' and 'gateway') from old to new mask and reboot.

To identify the mask of new router, I just fetched my local IP address on another device which was connected to same router.




---

[^1]:
[^2]:

