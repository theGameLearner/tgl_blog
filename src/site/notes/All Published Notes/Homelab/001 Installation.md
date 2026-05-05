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
- `AeG0og*123`
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

#### Homelab Topics Links

##### Understanding
Points to remember or understand:
- [[All Published Notes/Homelab/005 Impact of Changing ISP or Routers with Proxmox based Homelab\|005 Impact of Changing ISP or Routers with Proxmox based Homelab]]
- [[All Published Notes/Homelab/006 Connecting Proxmox to Wi-Fi\|006 Connecting Proxmox to Wi-Fi]]
- [[All Published Notes/Homelab/007 Giving names to all MAC address or Hardware\|007 Giving names to all MAC address or Hardware]]

##### Services
The services I run on my Homelab:
- NAS
	- I wanted a NAS option to store and retrieve files without having to connect and disconnect different Hard drives as needed.
	- [[All Published Notes/Homelab/003 Proxmox - Ubuntu LXC as NAS with personal HDDs\|003 Proxmox - Ubuntu LXC as NAS with personal HDDs]]
- DNS and VPN
	- options:
		- [Technitium](https://technitium.com/dns/): for people with good understanding and knowledge who need more control and have atleast medium level of understanding of how to manage a DNS
			- I did not find it easy to start with
		- [Pi-hole](https://pi-hole.net/): Easy solution for hobbyist to use directly without deep dive.
			- It did not feel as a ready to use solution
		- [adguard-home](https://adguard.com/en/adguard-home/overview.html): seems to be great option for ad blocking
			- My main focus is going to be accessing remote machine using the SSH and "RustDesk".
		- [Netbird](https://netbird.io/): Can serve as DNS as well as VPN and can also be self-hosted. This is a better alternative to TailScale as well as I have 5 email address in my family. 
			- 

##### Other
Other:
- [[All Published Notes/Homelab/004 Users and groups in Proxmox\|004 Users and groups in Proxmox]]
- 


---

[^1]:
[^2]:

