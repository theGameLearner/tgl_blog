---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/003-proxmox-hdd-as-nas/"}
---

created: 2026-03-22
updated: 2026-03-25

## Concept
To use a HDD as a NAS, we need to have the HDD plugged in and mounted, we have done this in [[All Published Notes/Homelab/002 First Setup in Proxmox#My Hard disks are not showing up\|My Drives are not showing up]] section.

Now, before we begin, we will use `df -h` to validate the drives are plugged in and mounted (`/dev/sdb1`, `/dev/sdc1`, `/dev/sdd1`):
```sh
root@pve:~# df -h
Filesystem            Size  Used Avail Use% Mounted on
udev                  7.5G     0  7.5G   0% /dev
tmpfs                 1.6G  1.6M  1.6G   1% /run
/dev/mapper/pve-root   68G  5.4G   59G   9% /
tmpfs                 7.6G   43M  7.5G   1% /dev/shm
efivarfs              128K   52K   72K  43% /sys/firmware/efi/efivars
tmpfs                 5.0M     0  5.0M   0% /run/lock
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            1022M  8.8M 1014M   1% /boot/efi
tmpfs                 7.6G     0  7.6G   0% /tmp
/dev/sdd1             932G  797G  135G  86% /mnt/games-setup
/dev/sdc1             932G  532G  400G  58% /mnt/media-hhd
/dev/fuse             128M   16K  128M   1% /etc/pve
/dev/sdb1             3.7T  220G  3.5T   6% /mnt/main-backup
tmpfs                 1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                 1.6G  4.0K  1.6G   1% /run/user/0
root@pve:~# 
```

We will now download and use a container image to define a 'LXC container' which will run ubuntu, and in this container, we will install samba and share the HDD through samba.

## Create a Container
#### Download a CT template
As we need a template to create containers, we will download it in 'local' which has 72 GB of space.
![008 Proxmox Download CT Template.png](/img/user/All%20Published%20Notes/Homelab/Images/008%20Proxmox%20Download%20CT%20Template.png)

Now we have 'ubuntu-24.04-standard' available as a base for any new container we want to make using Ubuntu as a base.
We wait till it is downloaded which is shown in a log after clicking 'Download' button.

#### Create a new CT
Click on 'Create CT' on top right

- hostname: `NAS-LXC`
- CT ID: `400`
- Password: `123TestN@$`
![009 Proxmox First CT.png](/img/user/All%20Published%20Notes/Homelab/Images/009%20Proxmox%20First%20CT.png)
Select the downloaded template from 'local' storage:
![010 Proxmox Choosing CT template.png](/img/user/All%20Published%20Notes/Homelab/Images/010%20Proxmox%20Choosing%20CT%20template.png)
Set the storage where this CT will reside and run from. As it is main CT for a NAS, we will run on 'local-lvm'. As Samba needs very little Disk size, but can increase in future, I am assigning it 8 GB Space.
![011 Proxmox CT Disks.png](/img/user/All%20Published%20Notes/Homelab/Images/011%20Proxmox%20CT%20Disks.png)
CPU cores: 4
Memory: 2048 MiB
Swap: 1024 MiB
Network:
Static IPv4 and DHCP IPv6
IPv4/CIDR: `192.168.31.200/24` (without `/24` the next button does not work)
Gateway (IPv4): 192.168.1.1
![012 Proxmox CT Network Settings.png](/img/user/All%20Published%20Notes/Homelab/Images/012%20Proxmox%20CT%20Network%20Settings.png)

DNS: leave blank
Final values:
![013 Proxmox CT confirm.png](/img/user/All%20Published%20Notes/Homelab/Images/013%20Proxmox%20CT%20confirm.png)

In my case, this gave a warning:
![014 Proxmox CT output.png](/img/user/All%20Published%20Notes/Homelab/Images/014%20Proxmox%20CT%20output.png)
The issue is that LXC container uses a newer version of systemd (255) that expects certain kernel features (like `CONFIG_USER_NS_UNPRIVILEGED`) to be available. Enabling **nesting** allows the container to run its own systemd instance properly.

> [!Note]
> In the world of Proxmox, **Nesting** allows a container (LXC) to act a bit more like a full Virtual Machine. It permits the container to use certain "virtual" filesystems (`procfs`, `sysfs`) that modern Linux distributions (like Ubuntu 24.04 with **Systemd 255**) require to start up their internal services.
##### What to do now
###### Check if the container was created
Check in web UI or command line if the container was created.
For web UI: Under the Datacenter, in pve, there should be "400 (NAS-LXC)"
For CLI:
```sh
root@pve:~# pct list
VMID       Status     Lock         Name
400        stopped                 NAS-LXC
root@pve:~#
```
###### Enable nesting
This is optional, if the container runs without issue, you don't need to do it, I am switching it on because I later plan on using Docker in same LXC later, and nesting is needed for Docker.
![015 Proxmox CT Nesting.png](/img/user/All%20Published%20Notes/Homelab/Images/015%20Proxmox%20CT%20Nesting.png)

you can enable nesting in CLI as well:
```sh
pct set 400 --features nesting=1
```
where 400 is the "CT ID" or "vmid" of this container.

#### Add the host mount points as bind‑mounts inside the container
edit the container's configuration file (replace `400` with your container ID):
```sh
root@pve:~# editor /etc/pve/lxc/400.conf
```

Now, we want to create a map for existing mounts in proxmox to mount points in the container, look at the bottom 3 lines:
```sh
root@pve:~# editor /etc/pve/lxc/400.conf
root@pve:~# cat /etc/pve/lxc/400.conf
arch: amd64
cores: 4
features: nesting=1
hostname: NAS-LXC
memory: 2048
net0: name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.1,hwaddr=BC:24:11:D6:4B:8B,ip=192.168.31.200/24,ip6=dhcp,type=veth
ostype: ubuntu
rootfs: local-lvm:vm-400-disk-0,size=8G
swap: 1024

mp0: /mnt/games-setup,mp=/mnt/hdd-games
mp1: /mnt/media-hhd,mp=/mnt/hdd-media
mp2: /mnt/main-backup,mp=/mnt/hdd-main-backup
root@pve:~# 
```

#### (Re)Start the Container and confirm it runs normally
We can start a container using CLI or by web UI, I will use web UI:
![016 Proxmox CT Start.png](/img/user/All%20Published%20Notes/Homelab/Images/016%20Proxmox%20CT%20Start.png)

if you want to use CLI to start a container(CT), use this:
```sh
pct start 400
```

After start, click on console and login as root with the password we set during container creation:
- username: `root`
- Password: `123TestN@$`

This should give you a prompt like: `root@NAS-LXC:~# `
![017 Proxmox CT login.png](/img/user/All%20Published%20Notes/Homelab/Images/017%20Proxmox%20CT%20login.png)

In my SSH connection where I am root@pve, I can enter the root of this container by the following:
```sh
root@pve:~# pct list
VMID       Status     Lock         Name                
400        stopped                 NAS-LXC             
root@pve:~# Started the container using web UI
root@pve:~# pct enter 400
root@NAS-LXC:~# pwd
/root
root@NAS-LXC:~# whoami
root
root@NAS-LXC:~# 
```

#### Install Samba
As we are on a new instance of machine, let us update it and install samba:
```sh
apt update && apt install samba -y
```
This will take time as all files will be downloaded to the container.

Create a new user in the container who will have all permissions as needed as well as add the same user to samba and set a password for it:
```sh
root@NAS-LXC:~# useradd -M -s /usr/sbin/nologin nasuser
root@NAS-LXC:~# smbpasswd -a nasuser
New SMB password:
Retype new SMB password:
Added user nasuser.
root@NAS-LXC:~# cat /var/lib/samba/private/passdb.tdb
TDB file
�L9�&INFO/version4r�T|�TL:�p�&NEXT_RID�,�OpC���zm�𥟰����� ���������������������� ���S�&INFO/minor_version8
�L9�&INFO/version4r�T|�TL:�p�&NEXT_RID�,Proot@NAS-LXC:~# or_version8&INFO/minor_version8
root@NAS-LXC:~# 
root@NAS-LXC:~# 
```

Effectively, your ubuntu has 2 users now, 'root' and 'nasuser' as well as we added the 'nasuser' to samba and set a password for it: `TestN@$123`.
Given that there is no user directory(`-M`) or interactive shell (`-s /usr/sbin/nologin`), the user acts as a utility user we can use to get our work done without worrying about others using this user name to SSH into our system.

Checking if the new user is added:
```sh
root@NAS-LXC:~# cut -d: -f1 /etc/passwd
root
daemon
bin
sys
sync
games
man
lp
mail
news
uucp
proxy
www-data
backup
list
irc
_apt
nobody
systemd-network
systemd-timesync
dhcpcd
messagebus
syslog
systemd-resolve
sshd
postfix
uuidd
tcpdump
nasuser
root@NAS-LXC:~# 
root@NAS-LXC:~# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
dhcpcd:x:100:65534:DHCP Client Daemon,,,:/usr/lib/dhcpcd:/bin/false
messagebus:x:101:101::/nonexistent:/usr/sbin/nologin
syslog:x:102:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin
sshd:x:103:65534::/run/sshd:/usr/sbin/nologin
postfix:x:104:105::/var/spool/postfix:/usr/sbin/nologin
uuidd:x:105:107::/run/uuidd:/usr/sbin/nologin
tcpdump:x:106:109::/nonexistent:/usr/sbin/nologin
nasuser:x:1000:1000::/home/nasuser:/usr/sbin/nologin
root@NAS-LXC:~#
```

#### Configure Samba
Edit the samba config so that my 'nasuser' can access everything and any guest can access(read-only) `/mnt/hdd-media` and `/mnt/hdd-games` but not `/mnt/hdd-main-backup`. The reason guests have read-only is because I do not want guests to be able to delete the files.

```sh
editor /etc/samba/smb.conf
```
This will open file in **nano**, as in ubuntu container, we have not installed fresh. Installation fails for me, so I will use nano here.

In case you need exact names of the drives in ubuntu, you can use `df -h` again:
```sh
root@NAS-LXC:~# df -h
Filesystem                        Size  Used Avail Use% Mounted on
/dev/mapper/pve-vm--400--disk--0  7.8G  999M  6.4G  14% /
/dev/sdd1                         932G  797G  135G  86% /mnt/hdd-games
/dev/sdc1                         932G  532G  400G  58% /mnt/hdd-media
/dev/sdb1                         3.7T  220G  3.5T   6% /mnt/hdd-main-backup
none                              492K  4.0K  488K   1% /dev
tmpfs                             7.6G     0  7.6G   0% /dev/shm
tmpfs                             3.1G  3.2M  3.1G   1% /run
tmpfs                             5.0M     0  5.0M   0% /run/lock
tmpfs                             1.6G  8.0K  1.6G   1% /run/user/0
root@NAS-LXC:~# 
```

I will explicitly use nasuser as a user for ubuntu NAS with authorization for write, guests can read media and games only.
```sh
root@NAS-LXC:~# editor /etc/samba/smb.conf
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

root@NAS-LXC:~# 
```

Now that the config file (`/etc/samba/smb.conf`) is updated, we need to restart samba for the changes in config to be loaded.
```sh
systemctl restart smbd
```

after restart, check it's status
```sh
systemctl status smbd
```
logs for reference:
```sh
root@NAS-LXC:~# systemctl status smbd
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/usr/lib/systemd/system/smbd.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-03-22 15:42:56 UTC; 2h 21min ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
   Main PID: 1827 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 3 (limit: 18264)
     Memory: 9.8M (peak: 10.8M)
        CPU: 100ms
     CGroup: /system.slice/smbd.service
             ├─1827 /usr/sbin/smbd --foreground --no-process-group
             ├─1830 "smbd: notifyd" .
             └─1831 "smbd: cleanupd "

Mar 22 15:42:56 NAS-LXC systemd[1]: Starting smbd.service - Samba SMB Daemon...
Mar 22 15:42:56 NAS-LXC (smbd)[1827]: smbd.service: Referenced but unset environment variable evaluates to an empty s>
Mar 22 15:42:56 NAS-LXC systemd[1]: Started smbd.service - Samba SMB Daemon.
root@NAS-LXC:~# 
root@NAS-LXC:~# systemctl stop smdb
Failed to stop smdb.service: Unit smdb.service not loaded.
root@NAS-LXC:~# systemctl stop smbd
root@NAS-LXC:~# systemctl status smbd
○ smbd.service - Samba SMB Daemon
     Loaded: loaded (/usr/lib/systemd/system/smbd.service; enabled; preset: enabled)
     Active: inactive (dead) since Sun 2026-03-22 18:04:54 UTC; 4s ago
   Duration: 2h 21min 58.533s
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
    Process: 1827 ExecStart=/usr/sbin/smbd --foreground --no-process-group $SMBDOPTIONS (code=killed, signal=TERM)
   Main PID: 1827 (code=killed, signal=TERM)
     Status: "smbd: ready to serve connections..."
        CPU: 104ms

Mar 22 15:42:56 NAS-LXC systemd[1]: Starting smbd.service - Samba SMB Daemon...
Mar 22 15:42:56 NAS-LXC (smbd)[1827]: smbd.service: Referenced but unset environment variable evaluates to an empty s>
Mar 22 15:42:56 NAS-LXC systemd[1]: Started smbd.service - Samba SMB Daemon.
Mar 22 18:04:54 NAS-LXC systemd[1]: Stopping smbd.service - Samba SMB Daemon...
Mar 22 18:04:54 NAS-LXC systemd[1]: smbd.service: Deactivated successfully.
Mar 22 18:04:54 NAS-LXC systemd[1]: Stopped smbd.service - Samba SMB Daemon.
root@NAS-LXC:~# systemctl start smbd
root@NAS-LXC:~# systemctl status smbd
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/usr/lib/systemd/system/smbd.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-03-22 18:05:23 UTC; 2s ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
    Process: 2239 ExecCondition=/usr/share/samba/is-configured smb (code=exited, status=0/SUCCESS)
   Main PID: 2242 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 3 (limit: 18264)
     Memory: 7.7M (peak: 8.1M)
        CPU: 42ms
     CGroup: /system.slice/smbd.service
             ├─2242 /usr/sbin/smbd --foreground --no-process-group
             ├─2245 "smbd: notifyd" .
             └─2246 "smbd: cleanupd "

Mar 22 18:05:23 NAS-LXC systemd[1]: Starting smbd.service - Samba SMB Daemon...
Mar 22 18:05:23 NAS-LXC (smbd)[2242]: smbd.service: Referenced but unset environment variable evaluates to an empty s>
Mar 22 18:05:23 NAS-LXC smbd[2242]: [2026/03/22 18:05:23.467549,  0] source3/smbd/server.c:1746(main)
Mar 22 18:05:23 NAS-LXC smbd[2242]:   smbd version 4.19.5-Ubuntu started.
Mar 22 18:05:23 NAS-LXC smbd[2242]:   Copyright Andrew Tridgell and the Samba Team 1992-2023
Mar 22 18:05:23 NAS-LXC systemd[1]: Started smbd.service - Samba SMB Daemon.
root@NAS-LXC:~# 
```

To verify, list all user accounts stored in the Samba database (SAM database):
```sh
root@NAS-LXC:~# pdbedit -L
nasuser:1000:
root@NAS-LXC:~#
```


#### Test connection
Now, Samba has access to drives and can share it, we need to test it.

##### Test in Container
Before we test access from outside, we may want to test the settings in local container, for this, we can use `smbclient` utility.
install it if not available:
```sh
apt update && apt install smbclient -y
```

```sh
root@NAS-LXC:~# # Test listing shares as guest
root@NAS-LXC:~# smbclient -L localhost -N

        Sharename       Type      Comment
        ---------       ----      -------
        hdd-main-backup Disk      
        hdd-media       Disk      
        hdd-games       Disk      
        IPC$            IPC       IPC Service (NAS-LXC server (Samba))
SMB1 disabled -- no workgroup available
root@NAS-LXC:~# # smbclient -L localhost -N
root@NAS-LXC:~# # Test listing shares as nasuser
root@NAS-LXC:~# smbclient -L localhost -U nasuser
Password for [WORKGROUP\nasuser]:

        Sharename       Type      Comment
        ---------       ----      -------
        hdd-main-backup Disk      
        hdd-media       Disk      
        hdd-games       Disk      
        IPC$            IPC       IPC Service (NAS-LXC server (Samba))
SMB1 disabled -- no workgroup available
root@NAS-LXC:~# # Test accessing a guest share
root@NAS-LXC:~# smbclient //localhost/hdd-media -N
Try "help" to get a list of possible commands.
smb: \> quit
root@NAS-LXC:~# # Test accessing the authenticated share
root@NAS-LXC:~# smbclient //localhost/hdd-main-backup -U nasuser
Password for [WORKGROUP\nasuser]:
Try "help" to get a list of possible commands.
smb: \> quit
root@NAS-LXC:~# # From inside the container, as nasuser (or as root), test if nasuser can write to the main backup share:
root@NAS-LXC:~# su - nasuser -s /bin/bash -c 'touch /mnt/hdd-main-backup/testfile && rm /mnt/hdd-main-backup/testfile && echo "Write works"'
su: warning: cannot change directory to /home/nasuser: No such file or directory
touch: cannot touch '/mnt/hdd-main-backup/testfile': Permission denied
root@NAS-LXC:~# ls -al /mnt/hdd-main-backup/
total 1924
drwxr-xr-x  2 root root 131072 Feb 12 12:59 '$RECYCLE.BIN'
drwxr-xr-x 16 root root 131072 Mar 22 11:05  .
drwxr-xr-x  5 root root   4096 Mar 22 15:30  ..
drwxr-xr-x  4 root root 131072 Mar 19 07:08  .Trash-1000
drwxr-xr-x 10 root root 131072 Jan  4 07:10  Documents
drwxr-xr-x  3 root root 131072 Jan  3 17:29  Education
drwxr-xr-x  6 root root 131072 Jan  4 07:28  Personal
drwxr-xr-x 10 root root 131072 Jan  3 17:28  Professional
drwxr-xr-x  4 root root 131072 Jan  3 12:40  System
drwxr-xr-x  2 root root 131072 Jan 16 02:41 'System Volume Information'
drwxr-xr-x  2 root root 131072 Mar 21 14:47  dump
drwxr-xr-x  2 root root 131072 Mar 21 14:47  images
drwxr-xr-x  2 root root 131072 Mar 21 14:47  import
drwxr-xr-x  2 root root 131072 Mar 21 14:47  private
drwxr-xr-x  2 root root 131072 Mar 21 14:47  snippets
drwxr-xr-x  4 root root 131072 Mar 21 14:47  template
root@NAS-LXC:~# 
```

We have read permission but maybe write is not available for now, we will solve this at a later moment, first I want to see if we can read the data as a guest in another machine.

##### Test in Linux
As my linux uses Nemo File Explorer, I open my file explorer, go to path, edit path to `smb://192.168.31.200/` and hit enter:
![018 NAS from Linux.png](/img/user/All%20Published%20Notes/Homelab/Images/018%20NAS%20from%20Linux.png)

This allows me to see mounted files without having to login using nasuser account. But opening any will ask me to authenticate:
![019 NAS from Linux 2.png](/img/user/All%20Published%20Notes/Homelab/Images/019%20NAS%20from%20Linux%202.png)

I can use Anonymous user for media and games. I can even play videos from my NAS without having a GPU in the proxmox machine.

#### Fixing write permission
Even if I authorize myself with samba password as nasuser and try to write new directories, my effort fails:
![020 NAS write error.png](/img/user/All%20Published%20Notes/Homelab/Images/020%20NAS%20write%20error.png)
Need to figure out why?
possible reasons:
- my proxmox has no write access as the HDD are all NTFS
- my container has errors in setup.
First, let me see if I can read and write to the HDDs as root user who has all permissions:
As *root* in *Proxmox* OS:
```sh
root@pve:~# df -h
Filesystem Size Used Avail Use% Mounted on
udev 7.5G 0 7.5G 0% /dev
tmpfs 1.6G 1.7M 1.6G 1% /run
/dev/mapper/pve-root 68G 5.5G 59G 9% /
tmpfs 7.6G 40M 7.5G 1% /dev/shm
efivarfs 128K 52K 72K 43% /sys/firmware/efi/efivars
tmpfs 5.0M 0 5.0M 0% /run/lock
tmpfs 1.0M 0 1.0M 0% /run/credentials/systemd-journald.service
/dev/sda2 1022M 8.8M 1014M 1% /boot/efi
tmpfs 7.6G 0 7.6G 0% /tmp
/dev/sdb1 932G 532G 400G 58% /mnt/media-hhd
/dev/sdd1 932G 797G 135G 86% /mnt/games-setup
/dev/fuse 128M 16K 128M 1% /etc/pve
/dev/sdc1 3.7T 220G 3.5T 6% /mnt/main-backup
tmpfs 1.0M 0 1.0M 0% /run/credentials/getty@tty1.service
tmpfs 1.6G 4.0K 1.6G 1% /run/user/0
root@pve:~# namei -om /mnt/media-hhd
f: /mnt/media-hhd
drwxr-xr-x root root /
drwxr-xr-x root root mnt
drwxr-xr-x root root media-hhd
root@pve:~# namei -om /mnt/media-games-setup
f: /mnt/media-games-setup
drwxr-xr-x root root /
drwxr-xr-x root root mnt
media-games-setup - No such file or directory
root@pve:~# namei -om /mnt/main-backup
f: /mnt/main-backup
drwxr-xr-x root root /
drwxr-xr-x root root mnt
drwxr-xr-x root root main-backup
root@pve:~#
```

as *root* user in *NAS-LXC*:
```sh
root@NAS-LXC:~# df -h
Filesystem Size Used Avail Use% Mounted on
/dev/mapper/pve-vm--400--disk--0 7.8G 1009M 6.4G 14% /
/dev/sdd1 932G 797G 135G 86% /mnt/hdd-games
/dev/sdb1 932G 532G 400G 58% /mnt/hdd-media
/dev/sdc1 3.7T 220G 3.5T 6% /mnt/hdd-main-backup
none 492K 4.0K 488K 1% /dev
tmpfs 7.6G 0 7.6G 0% /dev/shm
tmpfs 3.1G 3.2M 3.1G 1% /run
tmpfs 5.0M 0 5.0M 0% /run/lock
root@NAS-LXC:~# namei -om /mnt/hdd-games
f: /mnt/hdd-games
drwxr-xr-x root root /
drwxr-xr-x root root mnt
drwxr-xr-x root root hdd-games
root@NAS-LXC:~# namei -om /mnt/hdd-media
f: /mnt/hdd-media
drwxr-xr-x root root /
drwxr-xr-x root root mnt
drwxr-xr-x root root hdd-media
root@NAS-LXC:~# namei -om /mnt/hdd-main-backup
f: /mnt/hdd-main-backup
drwxr-xr-x root root /
drwxr-xr-x root root mnt
drwxr-xr-x root root hdd-main-backup
root@NAS-LXC:~# sudo -u nasuser test -rxw /mnt/hdd-media && echo "Write: YES" || echo "Write: NO"
test: missing argument after '/mnt/hdd-media'
Write: NO
root@NAS-LXC:~# sudo -u nasuser test -r /mnt/hdd-media && echo "Write: YES" || echo "Write: NO"
Write: YES
root@NAS-LXC:~# sudo -u nasuser test -r /mnt/hdd-games && echo "Write: YES" || echo "Write: NO"
Write: YES
root@NAS-LXC:~# sudo -u nasuser test -r /mnt/hdd-main-backup && echo "Write: YES" || echo "Write: NO"
Write: YES
root@NAS-LXC:~# sudo -u nasuser test -w /mnt/hdd-media && echo "Write: YES" || echo "Write: NO"
Write: NO
root@NAS-LXC:~# sudo -u nasuser test -w /mnt/hdd-games && echo "Write: YES" || echo "Write: NO"
Write: NO
root@NAS-LXC:~# sudo -u nasuser test -w /mnt/hdd-main-backup && echo "Write: YES" || echo "Write: NO"
Write: NO
root@NAS-LXC:~#
``` 

So, the 'Proxmox' OS's *'root' has read, write and execute permission* and for all other users (other than the owner), we only have read and execute permission. When no one but root(owner) has write permission, how can we give this permission inside 'NAS-LXC' to user 'nasuser'?

Right now, the 'nasuser' with UID 1000 is the user, but files are owned by root(UID 0), that is why 'nasuser' is blocked from writing.

> [!Note]
> The way Proxmox and '*Privileged* LXC container's permissions system' works is that the user ID and permissions are mapped 1:1, so even if names are different, if the user ID is a match, the permissions are matching.
> Meaning, if UID 1000 owns directory "path" in LXC container, any user in proxmox trying to access "path" as UID 1000 will be allowed as well.

To fix this, we have 2 choices:
- Use `force user = root` in Samba shares to make samba as root for all authenticated users.
- Remount the drives on Proxmox with appropriate `uid`/`gid` options (Make a user group in Proxmox and let them be able to write)

##### Make Samba as root to write
- fix permissions with Samba’s `force user` in configuration which let's samba act as root user and allows us to be able to write as needed 
	- this is a bad choice because anyone can use samba to write anything, making all permissions meaningless.
	- All authorized users are treated as root
```sh
[hdd-main-backup]
   path = /mnt/hdd-main-backup
   browseable = yes
   read only = no
   guest ok = no
   valid users = nasuser
   write list = nasuser
   force user = root          # <-- added
   create mask = 0777
   directory mask = 0777
[hdd-media]
   path = /mnt/hdd-media
   browseable = yes
   read only = no             # changed from 'yes' to allow writes for nasuser
   guest ok = yes
   write list = nasuser
   force user = root          # <-- added
   create mask = 0777
   directory mask = 0777
```

##### Make a user group in Proxmox and let them be able to write
Before anything is edited, let us understand the file type of all mounted drives in Proxmox:
```sh
root@pve:~# lsblk -f /dev/sdb1
NAME FSTYPE FSVER LABEL     UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sdb1 exfat  1.0   Movies TV 797F-72BF                             399.5G    57% /mnt/media-hhd
root@pve:~# lsblk -f /dev/sdc1
NAME FSTYPE FSVER LABEL       UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sdc1 exfat  1.0   4TB Main Us ED5E-0B40                               3.4T     6% /mnt/main-backup
root@pve:~# lsblk -f /dev/sdd1
NAME FSTYPE FSVER LABEL       UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sdd1 exfat  1.0   Games Setup EBDB-185F                             134.9G    86% /mnt/games-setup
root@pve:~# 
```
All 3 Hard drives are **exfat**, this is not good as we can not use `chown` or `chmod` with 'exfat' or 'NTFS' format.

> [!Note]
> **if it's exfat or NTFS:** We have to change the mount options in the Proxmox Host's `/etc/fstab` to tell it to "pretend" the whole drive is owned by 'nasuser'.

Now, we use 'mount' to understand the current owners of this drive:
```sh
root@pve:~# mount | grep exfat
/dev/sdb1 on /mnt/media-hhd type exfat (rw,nosuid,nodev,relatime,fmask=0022,dmask=0022,iocharset=utf8,errors=remount-ro)
/dev/sdd1 on /mnt/games-setup type exfat (rw,nosuid,nodev,relatime,fmask=0022,dmask=0022,iocharset=utf8,errors=remount-ro)
/dev/sdc1 on /mnt/main-backup type exfat (rw,nosuid,nodev,relatime,fmask=0022,dmask=0022,iocharset=utf8,errors=remount-ro)
root@pve:~# 
```

Based on above data, all 3 drives have 755(0777-0022) permission for files(fmask) and directories(dmask). This means only owner can write and anyone else can read and execute(enter) it. As the drives are exFat and unsupported by Linux, during mount, we gave them fake permissions which we need to correct now.

Now, we can create a group with write access and update in `/etc/fstab` of proxmox to give this new group write access to all mounted HDDs.

in proxmox, create a new user and group:
- User group name: 'NAS_User_Group' with GID 1000 (matching group id in NAS-LXC)
- user name: 'NAS_User' with UID 1000 (matching user id in NAS-LXC)
```sh
root@pve:~# groupadd -g 1000 NAS_User_Group
root@pve:~# useradd -u 1000 -g 1000 -M -s /usr/sbin/nologin NAS_User
root@pve:~# id NAS_User
uid=1000(NAS_User) gid=1000(NAS_User_Group) groups=1000(NAS_User_Group)
root@pve:~#
```
As the UID and GID are a match for this PAM user, all privileged LXC will have same or less permissions on all files based on UID and GID.

Now, let us change the `/etc/fstab` file.
Old values:
from: 
```sh
UUID=YOUR-UUID /mnt/media-hhd auto nosuid,nodev,nofail 0 2 
``` 
to: 
```sh
UUID=YOUR-UUID /mnt/media-hhd exfat nosuid,nodev,nofail,uid=1000,gid=1000,fmask=0000,dmask=0000 0 0 
```
here, we change the following:
- auto -> exfat: as we know the file format, we will use that for easy detection and avoiding any auto-detection issues.
- *nosuid*: Prevents a program on the HDD from running with "Root" privileges. This is a standard security best practice.
- *nodev*: Prevents the OS from interpreting character or block special devices on the partition (stops someone from creating a "fake" hard drive interface inside your data folder).
- *nofail*:Adding `nofail` tells Proxmox: "Try to mount this, but if it's not there, just keep booting." 
- *uid=1000,gid=1000*: Maps the "Owner" of the entire drive to your user who has UID 1000. (The newly created group and PAM user in Proxmox).
- **fmask/dmask=0000:** Gives full `777` permissions (Read/Write/Execute) to everyone on Proxmox, we let Ubuntu control the permissions in Samba, that is the only reason this is acceptable for the moment.

> [!Warning]
> If we ever decide to use internet to access data outside the LAN, this needs to be improved.

File Data:
```sh
root@pve:~# cat /etc/fstab
# <file system> <mount point> <type> <options> <dump> <pass>
/dev/pve/root / ext4 errors=remount-ro 0 1
UUID=2440-2998 /boot/efi vfat defaults 0 1
/dev/pve/swap none swap sw 0 0
proc /proc proc defaults 0 0

# Media Harddrive (sdb1)
UUID="797F-72BF" /mnt/media-hhd auto nosuid,nodev,nofail 0 2

# Games Setup (sdc1)
UUID="EBDB-185F" /mnt/games-setup auto nosuid,nodev,nofail 0 2

# Main Backup (sdd1)
UUID="ED5E-0B40" /mnt/main-backup auto nosuid,nodev,nofail 0 2
root@pve:~# # edit the file using fresh text editor
root@pve:~# fresh /etc/fstab
root@pve:~# cat /etc/fstab
# <file system> <mount point> <type> <options> <dump> <pass>
/dev/pve/root / ext4 errors=remount-ro 0 1
UUID=2440-2998 /boot/efi vfat defaults 0 1
/dev/pve/swap none swap sw 0 0
proc /proc proc defaults 0 0

# Media Harddrive (sdb1)
UUID="797F-72BF" /mnt/media-hhd exfat nosuid,nodev,nofail,uid=1000,gid=1000,fmask=0000,dmask=0000 0 0

# Games Setup (sdc1)
UUID="EBDB-185F" /mnt/games-setup exfat nosuid,nodev,nofail,uid=1000,gid=1000,fmask=0000,dmask=0000 0 0

# Main Backup (sdd1)
UUID="ED5E-0B40" /mnt/main-backup exfat nosuid,nodev,nofail,uid=1000,gid=1000,fmask=0000,dmask=0000 0 0
root@pve:~# 
```

Now, we can either restart the machine, so the updated `/etc/fstab` is used to load the data, or we can unmount and re-mount the drives. **To unmount and re-mount, we need to stop any container or VMs that might be using them**:
```sh
root@pve:~# systemctl daemon-reload
root@pve:~# umount -l /mnt/media-hhd
root@pve:~# umount -l /mnt/games-setup
root@pve:~# umount -l /mnt/main-backup
root@pve:~# mount -av
/                        : ignored
/boot/efi                : already mounted
none                     : ignored
/proc                    : already mounted
/mnt/media-hhd           : successfully mounted
/mnt/games-setup         : successfully mounted
/mnt/main-backup         : successfully mounted
root@pve:~# mount | grep exfat
/dev/sdb1 on /mnt/media-hhd type exfat (rw,nosuid,nodev,relatime,uid=1000,gid=1000,fmask=0000,dmask=0000,allow_utime=0022,iocharset=utf8,errors=remount-ro)
/dev/sdd1 on /mnt/games-setup type exfat (rw,nosuid,nodev,relatime,uid=1000,gid=1000,fmask=0000,dmask=0000,allow_utime=0022,iocharset=utf8,errors=remount-ro)
/dev/sdc1 on /mnt/main-backup type exfat (rw,nosuid,nodev,relatime,uid=1000,gid=1000,fmask=0000,dmask=0000,allow_utime=0022,iocharset=utf8,errors=remount-ro)
root@pve:~# 
root@pve:~# # restart the CT (NAS-LXC) using CLI or GUI
root@pve:~# 
```

Now, we can restart the CT (NAS-LXC) using CLI or GUI and then test again, without logging in as 'nasuser' in ubuntu, you should be able to read but you can only write if you are 'nasuser'.

Test result:
- as a guest user (unauthorized), went to edit path to `smb://192.168.31.200/` and hit enter:
	- can connect to 'hdd-games' and 'hdd-media' with connect button without a username or password.
	- cannot connect to 'hdd-main-backup' without a password
	- cannot move files in any of the drives as a guest user.
	- can play video files from 'hdd-media' in local video player 'Haruna' in personal linux machine.
![022 NAS guest user un-authorized action.png](/img/user/All%20Published%20Notes/Homelab/Images/022%20NAS%20guest%20user%20un-authorized%20action.png)
- as a authorized user(nasuser on Ubuntu):
	- can access 'hdd-main-backup' after registering as 'nasuser' and using password set in samba
	- can create, delete and move files in any of the drives.
![023 NAS authorize user login.png](/img/user/All%20Published%20Notes/Homelab/Images/023%20NAS%20authorize%20user%20login.png)

## Configure FileBowser in Ubuntu for browser access.
Next, if we do not want to use Nemo file explorer in the browser to login, we can use FileBowser to give a browser based access to the users in our LAN.
#incomplete 












---














---

[^1]:
[^2]:

