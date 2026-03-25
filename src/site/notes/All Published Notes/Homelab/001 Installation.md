---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/001-installation/"}
---

created: 2026-03-21
updated: 2026-03-21

I downloaded the proxmox ISO from the website and re-booted my system and selected the flash drive which had proxmox in the boot menu (F11 in MSI motherboard).

Steps done:
- select graphical interface for login.
- Accept license agreement
- The Proxmox Virtual Environment (PVE) only detects my 256 GB EVM SSD, so I did not have to choose a drive.
- In location and timezone settings:
	- Country: India
	- Timezone: Asia/Kolkata
	- Keyboard Layout: U.S. English
- In Administration Password and email address:
	- Password (root): `AeG0og*123`
	- Email: thegamelearner@gmail.com
- In Management Network Configuration
	- Management interface: 'nic0 - 34:5a:60:f9:47:63 (r8169)'
	- Hostname (FQDN): `pve.lan`
	- IP Address (CIDR) : `192.168.31.101`/`24`
	- Gateway: `192.168.31.1`
	- DNS Server: `192.168.31.1`
	- 'check' pin network interface names
- Confirm the summary
- Install with reboot option selected

This will take some time to install and reboot. Now we get a terminal screen with pve login, as my system is named pve(`pve.lan`), i will try to login using the credentials:
- root
- `AeG0og*124`
If the credentials are correct, the login screen changes to 
```sh
root@pve:~# 
```
which is shell after login. On this screen at the top you can see the IP and port which we can use to connect to this machine within the local area Network.

This means proxmox is loaded and can be used now.
open a browser and go to port 8006 of your proxmox IP address(as presented in login screen of this machine), so in my case `https://192.168.31.101:8006/`, the problem is that there are no certificates, so your browser may give a warning, ignore it and try to open the site.
The account is root until you create a new account:
- root
- `AeG0og*123`

After entering the system, you can now use proxmox as a headless server without a mouse, keyboard or a monitor.







---

[^1]:
[^2]:

