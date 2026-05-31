---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/005-impact-of-changing-isp-or-routers-with-proxmox-based-homelab/"}
---

created: 2026-04-03
updated: 2026-04-06

I had this experience so sharing the same here.
I changed my ISP from Jio to Airtel, meaning a new router and new setup. After switching on my proxmox machine, I was unable to log into `https://192.168.31.101:8006` which was set earlier. The reason was that my new router uses the mask `192.168.1.x` instead of `192.168.31.x`.

The Solution: I had to change the configurations in my proxmox as `root` user in `/etc/hosts`(`192.168.1.101 pve.lan pve`) and `/etc/network/interfaces`('address' and 'gateway') from old to new mask and reboot.
I also cannot resolve DNS without updating `/etc/resolv.conf` file which has old `192.168.31.1` mask:
```bash
root@pve:~# cat /etc/resolv.conf
search lan
nameserver 192.168.31.1
root@pve:~# 
```

To identify the mask of new router, I just fetched my local IP address on another device which was connected to same router.

To make this issue future-proof, I have 1 choice:
- Create a new **DHCP Pool** (The IPv4 range that a device can get on a router) and use this DHCP pool whenever you change the router. This will allow your Proxmox to always use the same pool and always be findable.
	- In Proxmox, update `/etc/network/interfaces` and `/etc/hosts` so they always use the new pool you have defined.
This allows me to find the proxmox on same IP and I need to only update the DHCP pool whenever I change my router in the future.



> [!Note]
> Currently my setup runs on `192.168.1.x` mask, and I have not updated the future proof idea mainly because airtel does not seem to allow users to play with subnet mask in their own homes.


I will now have to update my Proxmox data to use the new mask and any setting in old mask needs to be updated because routers do not give this flexibility.
- [[All Published Notes/Homelab/003 Proxmox - Ubuntu LXC as NAS with personal HDDs#How to change the configured IP for LXC\|Change Ubuntu LXC's configured IP]]
- 


---

[^1]:
[^2]:

