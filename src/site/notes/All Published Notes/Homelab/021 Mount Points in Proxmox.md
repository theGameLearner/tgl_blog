---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/021-mount-points-in-proxmox/"}
---

created: 2026-07-26
updated: 2026-07-26

# Mount points
In proxmox, the mount points define what hardware or data is mounted, and in which location.
I have recently bought a new HDD and want to backup my media and games in this HDD.
Now, I will have to find all mount points and change them as old mount points only work with existing HDDs which I will be removing.

### In Proxmox
#### Mount points in Proxmox
Let us find all mount points we can find in proxmox terminal.
##### storage points in Proxmox
Find Mount points connected to the Proxmox storage:
```sh
Sun Jul 26, 01:39:43 PM "~"|root@pve:# pvesm status
Name               Type     Status     Total (KiB)      Used (KiB) Available (KiB)        %
local               dir     active        71017632        10045588        57318824   14.15%
local-lvm       lvmthin     active       148086784        19147621       128939162   12.93%
main-backup         dir     active      3906862080       230722432      3676139648    5.91%

Sun Jul 26, 01:46:36 PM "~"|root@pve:# 
```

these are drives I have attached to Proxmox as storage for storing the LXC files, ISO, etc.

##### Volumes in Proxmox
Volumes visible in Proxmox UI:
```sh
Sun Jul 26, 01:46:36 PM "~"|root@pve:# cat /etc/pve/storage.cfg
dir: local
        path /var/lib/vz
        content import,iso,backup,vztmpl

lvmthin: local-lvm
        thinpool data
        vgname pve
        content rootdir,images

dir: main-backup
        path /mnt/main-backup
        content iso,rootdir
        prune-backups keep-all=1
        shared 0


Sun Jul 26, 01:46:44 PM "~"|root@pve:#
```

##### Active Disk Mount Points in Proxmox
Find all mounts in the machine that are of a specific format (not system default like `tmpfs` or `udev`):
```sh

Sun Jul 26, 01:46:44 PM "~"|root@pve:# findmnt -t ext4,exfat,xfs,vfat,zfs
TARGET               SOURCE               FSTYPE OPTIONS
/                    /dev/mapper/pve-root ext4   rw,relatime,errors=remount-ro
├─/mnt/games-setup   /dev/sdd1            exfat  rw,nosuid,nodev,relatime,uid=1000,gid=1000,fmask=0000,dmask=0000,allow_utime=0022,iocharset=utf8,errors=remount-ro
├─/mnt/media-hhd     /dev/sdb1            exfat  rw,nosuid,nodev,relatime,uid=1000,gid=1000,fmask=0000,dmask=0000,allow_utime=0022,iocharset=utf8,errors=remount-ro
├─/boot/efi          /dev/sda2            vfat   rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro
├─/mnt/main-backup   /dev/sdc1            exfat  rw,nosuid,nodev,relatime,uid=1000,gid=1000,fmask=0000,dmask=0000,allow_utime=0022,iocharset=utf8,errors=remount-ro
└─/mnt/entertainment /dev/sde2            ext4   rw,relatime

Sun Jul 26, 01:46:55 PM "~"|root@pve:# df -h -T | grep -v 'tmpfs\|udev'
Filesystem           Type      Size  Used Avail Use% Mounted on
/dev/mapper/pve-root ext4       68G  9.6G   55G  15% /
efivarfs             efivarfs  128K   52K   72K  43% /sys/firmware/efi/efivars
/dev/sdd1            exfat     932G  797G  135G  86% /mnt/games-setup
/dev/sdb1            exfat     932G  537G  396G  58% /mnt/media-hhd
/dev/sda2            vfat     1022M  8.8M 1014M   1% /boot/efi
/dev/sdc1            exfat     3.7T  221G  3.5T   6% /mnt/main-backup
/dev/fuse            fuse      128M   20K  128M   1% /etc/pve
/dev/sde2            ext4      1.8T  1.3T  410G  77% /mnt/entertainment

Sun Jul 26, 01:47:09 PM "~"|root@pve:# 
```

#### Mount points in Containers

Find all mount points defined in the configuration files of the containers:
```sh
Sun Jul 26, 01:56:08 PM "~"|root@pve:# grep -H '^mp[0-9]' /etc/pve/lxc/*.conf
/etc/pve/lxc/400.conf:mp0: /mnt/games-setup,mp=/mnt/hdd-games
/etc/pve/lxc/400.conf:mp1: /mnt/media-hhd,mp=/mnt/hdd-media
/etc/pve/lxc/400.conf:mp2: /mnt/main-backup,mp=/mnt/hdd-main-backup
/etc/pve/lxc/401.conf:mp0: /mnt/main-backup/homelab_backups/vikunja_lxc_backup,mp=/vikunja_backup
/etc/pve/lxc/401.conf:mp1: /mnt/main-backup/homelab/apps,mp=/apps
/etc/pve/lxc/401.conf:mp2: /mnt/games-setup,mp=/mnt/hdd-games
/etc/pve/lxc/401.conf:mp3: /mnt/media-hhd,mp=/mnt/hdd-media
/etc/pve/lxc/401.conf:mp4: /mnt/main-backup,mp=/mnt/hdd-main-backup

Sun Jul 26, 01:56:09 PM "~"|root@pve:# 
```

So, I have 2 containers, first has 3 mount points whereas second has 5 mount points.

We can find the same for each LXC:
```sh
Sun Jul 26, 02:06:06 PM "/"|root@pve:# pct config 400 | grep '^mp'
mp0: /mnt/games-setup,mp=/mnt/hdd-games
mp1: /mnt/media-hhd,mp=/mnt/hdd-media
mp2: /mnt/main-backup,mp=/mnt/hdd-main-backup

Sun Jul 26, 02:06:14 PM "/"|root@pve:# pct config 401 | grep '^mp'
mp0: /mnt/main-backup/homelab_backups/vikunja_lxc_backup,mp=/vikunja_backup
mp1: /mnt/main-backup/homelab/apps,mp=/apps
mp2: /mnt/games-setup,mp=/mnt/hdd-games
mp3: /mnt/media-hhd,mp=/mnt/hdd-media
mp4: /mnt/main-backup,mp=/mnt/hdd-main-backup

Sun Jul 26, 02:06:19 PM "/"|root@pve:# 
```





















---

[^1]: 
[^2]: 

