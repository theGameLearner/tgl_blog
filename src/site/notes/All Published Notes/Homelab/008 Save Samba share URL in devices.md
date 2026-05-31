---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/008-save-samba-share-url-in-devices/"}
---

created: 2026-04-18
updated: 2026-04-18

## Save Samba share URL in Nemo browser
If we are not connected to NetBird, we can use the local IP address of Proxmox host to access the "Privileged" Container(unprivileged container-> No) and get the NAS files through Samba. 
If the Proxmox host machine's LXC has the ip address `192.168.1.200/24`, the samba is available at `smb://192.168.1.200/`. This includes the config file in LXC which is used to define the settings, currently my settings looked like this:
```sh title:/etc/samba/smb.conf
root@NAS-LXC:~# cat /etc/samba/smb.conf
[global]
   workgroup = WORKGROUP
   server string = %h server (Samba)
   netbios name = NAS-LXC
   security = user
   map to guest = Bad User
   guest account = nobody
   log file = /var/log/samba/log.%m
   max log size = 1000
   socket options = TCP_NODELAY

[hdd-main-backup]
   path = /mnt/hdd-main-backup
   browseable = yes
   read only = no
   guest ok = no
   create mask = 0777
   directory mask = 0777
   valid users = nasuser
   write list = nasuser

[hdd-media]
   path = /mnt/hdd-media
   browseable = yes
   read only = yes
   guest ok = yes
   create mask = 0777
   directory mask = 0777
   write list = nasuser

[hdd-games]
   path = /mnt/hdd-games
   browseable = yes
   read only = yes
   guest ok = yes
   create mask = 0777
   directory mask = 0777
   write list = nasuser
```

If we are connected, we can use the URL with the assigned host name and connect to samba as well, but apparently, being connected to host will not work directly.

There are 2 steps to fix this, but I am hoping only step 1 will be enough.
- As LXC with my NAS does not have NetBird installed, direct access will not be possible. So I will install and configure NetBird on this LXC to test.
	- After config, the NetBird IP address is: `100.77.223.52(nas-lxc.netbird.cloud)` 
	- So the new samba URL should be: `smb://nas-lxc.netbird.cloud`
	- Attempting this did work when my proxmox host, the NAS LXC and my main Pc which I was using to access the NAS all were connected to NetBird.
	- Using NetBird does not stop normal IP address connection.
- If this is not enough, add lines to your `/etc/samba/smb.conf` file's 'global' section to allow hosts in a specific ip mask range like: `hosts allow = 127. 192.168.1. 100.77.` which whitelists all requests from these subnet masks.

## Save Samba share URL in Android
Ensure the Netbird is connected on the device where we want to use this Samba files.
Install an app that allows to create and save SMB urls (like CX File Explorer). Add a new SMB connection with the same local cloud URL `smb://nas-lxc.netbird.cloud`, this works (the movie files cannot be streamed easily on phone which might be an issue with the format of video or device compatibility). Alternatively, we can try to use the actual IP address instead of the Netbird custom name(`smb://192.168.1.200/`) and we can still access the files.



---

[^1]:
[^2]:

