---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/022-changing-the-harddrive-in-homelab/"}
---

created: 2026-07-26
updated: 2026-07-26


### Current Situation
I had connected 3 hard disk to my Proxmox server(2 HDD of 1 TB, 1 HDD of 1 TB), I am now trying to add another 2 TB HDD which will replace the 2 1TB hard disk.

```sh
Fri Jul 24, 07:29:17 PM "~"|root@pve:# lsblk
NAME                         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                            8:0    0 238.5G  0 disk 
├─sda1                         8:1    0  1007K  0 part 
├─sda2                         8:2    0     1G  0 part /boot/efi
└─sda3                         8:3    0 237.5G  0 part 
  ├─pve-swap                 252:0    0     8G  0 lvm  [SWAP]
  ├─pve-root                 252:1    0  69.4G  0 lvm  /
  ├─pve-data_tmeta           252:2    0   1.4G  0 lvm  
  │ └─pve-data-tpool         252:4    0 141.2G  0 lvm  
  │   ├─pve-data             252:5    0 141.2G  1 lvm  
  │   ├─pve-vm--400--disk--0 252:6    0     8G  0 lvm  
  │   └─pve-vm--401--disk--0 252:7    0    24G  0 lvm  
  └─pve-data_tdata           252:3    0 141.2G  0 lvm  
    └─pve-data-tpool         252:4    0 141.2G  0 lvm  
      ├─pve-data             252:5    0 141.2G  1 lvm  
      ├─pve-vm--400--disk--0 252:6    0     8G  0 lvm  
      └─pve-vm--401--disk--0 252:7    0    24G  0 lvm  
sdb                            8:16   0 931.5G  0 disk 
└─sdb1                         8:17   0 931.5G  0 part /mnt/media-hhd
sdc                            8:32   0   3.6T  0 disk 
└─sdc1                         8:33   0   3.6T  0 part /mnt/main-backup
sdd                            8:48   0 931.5G  0 disk 
└─sdd1                         8:49   0 931.5G  0 part /mnt/games-setup

Fri Jul 24, 07:32:30 PM "~"|root@pve:# blkid /dev/sdb1 /dev/sdc1 /dev/sdd1
/dev/sdb1: LABEL="Movies TV" UUID="797F-72BF" BLOCK_SIZE="512" TYPE="exfat" PARTLABEL="My Passport" PARTUUID="2c14786a-4da1-4221-a63e-f7a4cd24fcdc"
/dev/sdc1: LABEL="4TB Main Us" UUID="ED5E-0B40" BLOCK_SIZE="512" TYPE="exfat" PARTLABEL="primary" PARTUUID="8bd2f2a4-a886-44ec-aebb-25fe2f41fa93"
/dev/sdd1: LABEL="Games Setup" UUID="EBDB-185F" BLOCK_SIZE="512" TYPE="exfat" PARTUUID="935662a0-01"

Fri Jul 24, 07:34:19 PM "~"|root@pve:# cat -n /etc/fstab
     1  # <file system> <mount point> <type> <options> <dump> <pass>
     2  /dev/pve/root / ext4 errors=remount-ro 0 1
     3  UUID=2440-2998 /boot/efi vfat defaults 0 1
     4  /dev/pve/swap none swap sw 0 0
     5  proc /proc proc defaults 0 0
     6
     7  # Media Harddrive (sdb1)
     8  UUID="797F-72BF" /mnt/media-hhd exfat nosuid,nodev,nofail,uid=1000,gid=1000,fmask=0000,dmask=0000 0 0
     9
    10  # Games Setup (sdc1)
    11  UUID="EBDB-185F" /mnt/games-setup exfat nosuid,nodev,nofail,uid=1000,gid=1000,fmask=0000,dmask=0000 0 0
    12
    13  # Main Backup (sdd1)
    14  UUID="ED5E-0B40" /mnt/main-backup exfat nosuid,nodev,nofail,uid=1000,gid=1000,fmask=0000,dmask=0000 0 0
```

The hard disks are also in use by the LXCs, so I need to update everything.

### Adding the Hard disk
##### The hard disk is added in ext4 format
Connect the Hard disk, unmount the drive, format the drive in ext4 format, make a new mount point, and mount the drive:
In my main computer, the drive is connected and after using `lsblk` I found the label to be `/dev/sdc2`.

Un-mount the drive:
```sh
sudo umount /dev/sdc2
```

Format the drive:
```sh
sudo mkfs.ext4 -L "Rish Main Backup" /dev/sdc2
```

Create the mount point and mount the drive
```sh
sudo mkdir -p "/media/thegamelearner/Rish Main Backup"
sudo mount /dev/sdc2 "/media/thegamelearner/Rish Main Backup"
```

##### Add the new drive to Proxmox
Plug the hard disk into the machine running Proxmox.
use `blkid` to find it's label:
```sh
Sat Jul 25, 12:27:38 AM "~"|root@pve:# blkid
/dev/mapper/pve-root: UUID="107739a9-5999-45fa-a2e2-c2b31b1eb414" BLOCK_SIZE="4096" TYPE="ext4"
/dev/sdd1: LABEL="Games Setup" UUID="EBDB-185F" BLOCK_SIZE="512" TYPE="exfat" PARTUUID="935662a0-01"
/dev/sdb1: LABEL="Movies TV" UUID="797F-72BF" BLOCK_SIZE="512" TYPE="exfat" PARTLABEL="My Passport" PARTUUID="2c14786a-4da1-4221-a63e-f7a4cd24fcdc"
/dev/mapper/pve-swap: UUID="a47be2ea-593d-4fcc-bbc5-0bf89ba0d098" TYPE="swap"
/dev/sdc1: LABEL="4TB Main Us" UUID="ED5E-0B40" BLOCK_SIZE="512" TYPE="exfat" PARTLABEL="primary" PARTUUID="8bd2f2a4-a886-44ec-aebb-25fe2f41fa93"
/dev/sda2: UUID="2440-2998" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="8a3ce2ce-cfc4-435a-8841-70187164f065"
/dev/sda3: UUID="zLsPt7-MXYW-Z8FV-MU3N-MgFC-R2Lg-sOjJPb" TYPE="LVM2_member" PARTUUID="0e29bd52-e734-4da9-b5ef-07b31602a1fe"
/dev/mapper/pve-vm--400--disk--0: UUID="997488d5-4c5a-409b-86f8-eb6811adf593" BLOCK_SIZE="4096" TYPE="ext4"
/dev/sde2: LABEL="Rish Main Backup" UUID="e0c294c8-ec80-45bb-8e19-13ef537bf24d" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="Basic data partition" PARTUUID="9d8013b5-3467-49d0-91ab-8d1dad4074e5"
/dev/mapper/pve-vm--401--disk--0: UUID="87a1a2be-dd44-4029-8439-d0181e220619" BLOCK_SIZE="4096" TYPE="ext4"
/dev/sde1: PARTLABEL="Microsoft reserved partition" PARTUUID="446e7771-807b-4af5-996c-96212a78c3c1"
/dev/sda1: PARTUUID="1fd0a67e-b7f6-4079-abef-b256b81f7937"
```
The label for `Rish Main Backup` is `/dev/sde2` with UUID `e0c294c8-ec80-45bb-8e19-13ef537bf24d`.

Use the UUID and add it to fstab file, as we want the HDD to use the same mount points even if the HDD will be disconnected or re-attacked in a different port.

Make the new mount point and update the fstab file:
```sh
Sat Jul 25, 12:27:43 AM "~"|root@pve:# mkdir -p /mnt/entertainment

Sat Jul 25, 12:29:22 AM "~"|root@pve:# fresh /etc/fstab

A new version of fresh is available: 0.2.17 -> 0.4.5
Download from: https://github.com/sinelaw/fresh/releases/tag/v0.4.5


Sat Jul 25, 12:31:37 AM "~"|root@pve:# cat -n /etc/fstab
     1  # <file system> <mount point> <type> <options> <dump> <pass>
     2  /dev/pve/root / ext4 errors=remount-ro 0 1
     3  UUID=2440-2998 /boot/efi vfat defaults 0 1
     4  /dev/pve/swap none swap sw 0 0
     5  proc /proc proc defaults 0 0
     6
     7  # Media Harddrive (sdb1)
     8  UUID="797F-72BF" /mnt/media-hhd exfat nosuid,nodev,nofail,uid=1000,gid=1000,fmask=0000,dmask=0000 0 0
     9
    10  # Games Setup (sdc1)
    11  UUID="EBDB-185F" /mnt/games-setup exfat nosuid,nodev,nofail,uid=1000,gid=1000,fmask=0000,dmask=0000 0 0
    12
    13  # Main Backup (sdd1)
    14  UUID="ED5E-0B40" /mnt/main-backup exfat nosuid,nodev,nofail,uid=1000,gid=1000,fmask=0000,dmask=0000 0 0
    15
    16  # New 2 TB HDD (backup)
    17  UUID=e0c294c8-ec80-45bb-8e19-13ef537bf24d /mnt/entertainment ext4 defaults,nofail 0 2
    18

Sat Jul 25, 12:31:43 AM "~"|root@pve:# 
```
Here, because the format changed from exfat(windows compatible without linux permissions) to ext4, we also have to change the options and pass:
- Options
	- Removed
		- `nosuid`: ignores `setuid`/`setgid` security bits.
		- `nodev`: blocks special device files (like `/dev/` nodes).
		- `uid` and `gid` options are for *FAT/exFAT* filesystems only.
		- `fmask` & `dmask` *ONLY work on non-Linux filesystems* (exFAT, FAT32, NTFS). 
			- Because exFAT doesn't support Linux permissions natively, Linux uses `fmask` (file mask) and `dmask` (directory mask) at mount time to fake user permissions across the whole drive.
		- `0 0`: This is dump and FSCK Pass (Filesystem Check Priority)
			- 'dump': this is no longer used as I read, so we use 0 as default option
			- 'FSCK pass': 0 means "Do not check at boot time" (used for exFAT, swap, or network mounts).
	- Added
		- `defaults`: These are a set of default options for ext4 based files
			- `rw` — Mounts the drive as readable and writable.
			- `suid` — Allows Set Owner User ID (`setuid`) execution bits (opposite of `nosuid`).
			- `dev` — Allows character/block special devices to work on the filesystem (opposite of `nodev`).
			- `exec` — Allows binaries/scripts on the drive to be executed directly.
			- `auto` — Mounts automatically when `mount -a` or system boot runs.
			- `nouser` — Only `root` (or `sudo`) can explicitly mount/unmount the partition manually.
			- `async` — Disk I/O is asynchronous (buffered in RAM for maximum performance).
		- `0 2`: This is dump and FSCK Pass (Filesystem Check Priority)
			- 'dump': this is no longer used as I read, so we use 0 as default option
			- 'FSCK pass': 2 means "Secondary local filesystems". These are checked _after_ the root drive.
	- Retained
		- `nofail`: Ensures that system can start even if the drive is dis-connected


Mount all drives(`mount -a`):
```sh
Sat Jul 25, 12:40:27 AM "~"|root@pve:# df -h
Filesystem            Size  Used Avail Use% Mounted on
udev                  7.5G     0  7.5G   0% /dev
tmpfs                 1.6G  2.1M  1.6G   1% /run
/dev/mapper/pve-root   68G  9.6G   55G  15% /
tmpfs                 7.6G   37M  7.5G   1% /dev/shm
efivarfs              128K   52K   72K  43% /sys/firmware/efi/efivars
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
tmpfs                 5.0M     0  5.0M   0% /run/lock
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
tmpfs                 7.6G     0  7.6G   0% /tmp
/dev/sdd1             932G  797G  135G  86% /mnt/games-setup
/dev/sdb1             932G  537G  396G  58% /mnt/media-hhd
/dev/sda2            1022M  8.8M 1014M   1% /boot/efi
/dev/sdc1             3.7T  221G  3.5T   6% /mnt/main-backup
/dev/fuse             128M   20K  128M   1% /etc/pve
tmpfs                 1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                 1.6G  4.0K  1.6G   1% /run/user/0

Sat Jul 25, 12:40:29 AM "~"|root@pve:# mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.

Sat Jul 25, 12:40:34 AM "~"|root@pve:# df -h
Filesystem            Size  Used Avail Use% Mounted on
udev                  7.5G     0  7.5G   0% /dev
tmpfs                 1.6G  2.1M  1.6G   1% /run
/dev/mapper/pve-root   68G  9.6G   55G  15% /
tmpfs                 7.6G   37M  7.5G   1% /dev/shm
efivarfs              128K   52K   72K  43% /sys/firmware/efi/efivars
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
tmpfs                 5.0M     0  5.0M   0% /run/lock
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
tmpfs                 7.6G     0  7.6G   0% /tmp
/dev/sdd1             932G  797G  135G  86% /mnt/games-setup
/dev/sdb1             932G  537G  396G  58% /mnt/media-hhd
/dev/sda2            1022M  8.8M 1014M   1% /boot/efi
/dev/sdc1             3.7T  221G  3.5T   6% /mnt/main-backup
/dev/fuse             128M   20K  128M   1% /etc/pve
tmpfs                 1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                 1.6G  4.0K  1.6G   1% /run/user/0
/dev/sde2             1.8T   12K  1.7T   1% /mnt/entertainment

Sat Jul 25, 12:40:42 AM "~"|root@pve:# 
```

Now the drive is mounted as `/mnt/entertainment` directory in the machine.

##### Change HDD owner
We will ensure the new drive user is same as all other drives.
Find current drive owners:
```sh
Sat Jul 25, 12:54:51 AM "~"|root@pve:# ls -al /mnt/
total 400
drwxr-xr-x  7 root     root             4096 Jul 25 00:29 .
drwxr-xr-x 18 root     root             4096 Mar 21 17:05 ..
drwxr-xr-x  2 NAS_User NAS_User_Group   4096 Jul 23 22:59 entertainment
drwxrwxrwx  9 NAS_User NAS_User_Group 131072 Jun 28 13:33 games-setup
drwxrwxrwx 18 NAS_User NAS_User_Group 131072 Jul  5 01:19 main-backup
drwxrwxrwx  9 NAS_User NAS_User_Group 131072 Jul  5 01:19 media-hhd
drwxr-xr-x  2 root     root             4096 May 16 17:59 temp-lxc

Sat Jul 25, 12:54:57 AM "~"|root@pve:# 
```

Using fstab, we know that the drives have uid and gid of 1000, so `NAS_User` has user id of 1000 and `NAS_User_Group` has group id of 1000. We can also validate this using `getent passwd`.

If the owners are different, we can change the owner of `entertainment` to be same as others:
```sh
chown -R NAS_User:NAS_User_Group /mnt/entertainment
```

### Copy the data to the new drive
we can now copy data using `rsync` so we do not miss any hidden folder:
```sh
rsync -avP "/mnt/games-setup/Games Setup/" "/mnt/entertainment/Games Setup/" 
rsync -avP "/mnt/media-hhd/Media/" "/mnt/entertainment/Media/"
```

Example:
```sh
Sat Jul 25, 01:00:23 AM "~"|root@pve:# rsync -avP "/mnt/games-setup/Games Setup/" "/mnt/entertainment/Games Setup/"
sending incremental file list
created directory /mnt/entertainment/Games Setup
./
Windows/
Windows/Counter Strike 1.6/
...
Windows/unreal tournament/UnrealTournament/Web/plaintext/defaults_settings.uhtm
          1,316 100%   10.28kB/s    0:00:00 (xfr#14553, to-chk=3/14933)
Windows/unreal tournament/UnrealTournament/Web/plaintext/menu.uhtm
            639 100%    4.95kB/s    0:00:00 (xfr#14554, to-chk=2/14933)
Windows/unreal tournament/UnrealTournament/Web/plaintext/message.uhtm
            190 100%    1.45kB/s    0:00:00 (xfr#14555, to-chk=1/14933)
Windows/unreal tournament/UnrealTournament/Web/plaintext/root.uhtm
            359 100%    2.68kB/s    0:00:00 (xfr#14556, to-chk=0/14933)

sent 857,456,713,015 bytes  received 279,354 bytes  54,963,430.17 bytes/sec
total size is 853,807,898,671  speedup is 1.00
rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1338) [sender=3.4.1]

Sat Jul 25, 05:20:24 AM "~"|root@pve:#
```
and 
```sh
Sat Jul 25, 03:16:17 AM "~"|root@pve:# rsync -avP "/mnt/media-hhd/Media/" "/mnt/entertainment/Media/"
sending incremental file list
created directory /mnt/entertainment/Media
./
Anime/
Anime/Extra/
Anime/Extra/SAO/
Anime/Extra/SAO/SAO S01 Opening Theme Song.mp4
      6,835,181 100%  124.76MB/s    0:00:00 (xfr#1, ir-chk=1001/1015)
Anime/Movie/
Anime/Movie/Megamind (2010)/
Anime/Movie/Megamind (2010)/Megamind 2010.720p.BrRip.x264.YIFY.mp4
...
Wallpapers/BG Wallpeper 03.jpg
        379,169 100%  488.50kB/s    0:00:00 (xfr#2634, to-chk=2/2873)
Wallpapers/BG Wallpeper 04.jpg
         82,720 100%  106.43kB/s    0:00:00 (xfr#2635, to-chk=1/2873)
Wallpapers/Kirito 001.jpg
         33,257 100%   42.79kB/s    0:00:00 (xfr#2636, to-chk=0/2873)

sent 574,591,779,215 bytes  received 51,890 bytes  22,135,869.44 bytes/sec
total size is 574,451,303,133  speedup is 1.00

Sat Jul 25, 10:29:11 AM "~"|root@pve:# 
```

Depending on files to be copied, and the size, this can take very long time. For example, copying my games took 04:20 hours.

We can then run verification:
```sh

Sat Jul 25, 10:29:11 AM "~"|root@pve:# rsync -avnc "/mnt/media-hhd/Media/" "/mnt/entertainment/Media/"
sending incremental file list

sent 167,695 bytes  received 268 bytes  11.65 bytes/sec
total size is 574,451,303,133  speedup is 3,420,106.23 (DRY RUN)

Sat Jul 25, 03:22:48 PM "~"|root@pve:# 
```

If the list is empty, that means the copy completed successfully.
If there are errors, we get files, and we can copy them individually:
```sh
Sat Jul 25, 07:38:05 AM "~"|root@pve:# rsync -avnc "/mnt/games-setup/Games Setup/" "/mnt/entertainment/Games Setup/"

sending incremental file list

Windows/Mark of the Ninja - Remastered [FitGirl Repack]/fg-01.bin

  

sent 508,821 bytes  received 490 bytes  19.70 bytes/sec

total size is 853,807,898,671  speedup is 1,676,397.92 (DRY RUN)

  

Sat Jul 25, 02:49:11 PM "~"|root@pve:# rsync -avc "/mnt/games-setup/Games Setup/Windows/Mark of the Ninja - Remastered [FitGirl Repack]/fg-01.bin" "/mnt/entertainment/Games Setup/Windows/Mark of the Ninja - Remastered [FitGirl Repack]/fg-01.bin"
```

Then you can confirm it again:
```sh
Sun Jul 26, 11:03:54 AM "~"|root@pve:# ls -al "/mnt/games-setup/Games Setup/Windows/Mark of the Ninja - Remastered [FitGirl Repack]/fg-01.bin" "/mnt/entertainment/Games Setup/Windows/Mark of the Ninja - Remastered [FitGirl Repack]/fg-01.bin"
-rwxrwxrwx 1 NAS_User NAS_User_Group 3438658964 Jul 25 03:17 '/mnt/entertainment/Games Setup/Windows/Mark of the Ninja - Remastered [FitGirl Repack]/fg-01.bin'
-rwxrwxrwx 1 NAS_User NAS_User_Group 3438658964 Mar  9  2020 '/mnt/games-setup/Games Setup/Windows/Mark of the Ninja - Remastered [FitGirl Repack]/fg-01.bin'

Sun Jul 26, 11:04:14 AM "~"|root@pve:# 
```


### Add the HDD to Proxmox
You can add the HDD to proxmox only if it is used for ISO, memory or something similar inside Proxmox.

#### Should You add the HDD to Proxmox
Datacenter's Storage is specifically the Proxmox Storage Manager, designed to manage volumes for VMs, CT templates, ISOs, backups, and disk images. But there's no benefit to adding it to `pvesm` if you aren't using those Proxmox-specific content types.

|Your Goal|Should you use `pvesm`?|What to do instead|
|---|---|---|
|**Store files/media (NAS dump)** inside LXCs|❌ **No**|Mount the drive, then Bind Mount (`mp0:`) to the LXC.|
|**Store files/media (NAS dump)** over the network|❌ **No**|Mount the drive, then share it via SMB/NFS from the host.|
|**Store VM virtual disks (`.qcow2/.raw`)**|✅ **Yes**|Use `pvesm add dir` (and point it to a subfolder).|
|**Store LXC templates/ISOs/Backups**|✅ **Yes**|Use `pvesm add dir` (but you already said you don't use these).|

#### Add as a Directory
To add the hard drive to proxmox as a directory, you can use the GUI but it needs a type of Directory that you are adding.
We have the following options:
- Disk Image
- ISO Image
- Container Template
- Backup
- Container
- Snippets
- Import 

This is useful when you need to store the ISO or other data used in proxmox inside the HDD.
You can add a new directory to proxmox using the GUI or CLI.

Using CLI:
```sh
pvesm add dir Entertainment --path /mnt/entertainment --content images
```
creates a new directory called "Entertainment" for mount point "/mnt/entertainment" for 'Images' as content type.

Using GUI:
Open the GUI application on a browser: `https://192.168.1.101:8006/`
Login
Above PVE(node), you have 'Datacenter'.
Go to Datacenter -> Storage -> then click on Add.

### Add the HDD as Mount points
Let us edit the current existing mount points to use the new HDD as data points.
Find all current mount points after shutting down the existing LXCs:
```sh
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

Let us now update the conf file for both LXC to use the new '/mnt/entertainment' drive instead of games and media.

We open the configuration file in `fresh` terminal editor.
```sh
Sun Jul 26, 02:07:44 PM "/"|root@pve:# fresh /etc/pve/lxc/400.conf

A new version of fresh is available: 0.2.17 -> 0.4.5
Download from: https://github.com/sinelaw/fresh/releases/tag/v0.4.5


Sun Jul 26, 02:12:04 PM "/"|root@pve:# cat -n /etc/pve/lxc/400.conf
     1  arch: amd64
     2  cores: 4
     3  features: nesting=1
     4  hostname: NAS-LXC
     5  memory: 2048
     6  mp0: /mnt/entertainment/Games Setup,mp=/mnt/hdd-games
     7  mp1: /mnt/entertainment/Media,mp=/mnt/hdd-media
     8  mp2: /mnt/main-backup,mp=/mnt/hdd-main-backup
     9  net0: name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.1,hwaddr=BC:24:11:D6:4B:8B,ip=192.168.1.200/24,ip6=dhcp,type=veth
    10  onboot: 1
    11  ostype: ubuntu
    12  rootfs: local-lvm:vm-400-disk-0,size=8G
    13  swap: 1024
    14  tags: nas

Sun Jul 26, 02:12:12 PM "/"|root@pve:# 
```


Similarly, edit the conf for 401:
```sh

Sun Jul 26, 02:12:12 PM "/"|root@pve:# cat -n /etc/pve/lxc/401.conf
     1  arch: amd64
     2  cores: 4
     3  features: fuse=1,keyctl=1,nesting=1
     4  hostname: DockerHost
     5  memory: 2048
     6  mp0: /mnt/main-backup/homelab_backups/vikunja_lxc_backup,mp=/vikunja_backup
     7  mp1: /mnt/main-backup/homelab/apps,mp=/apps
     8  mp2: /mnt/games-setup,mp=/mnt/hdd-games
     9  mp3: /mnt/media-hhd,mp=/mnt/hdd-media
    10  mp4: /mnt/main-backup,mp=/mnt/hdd-main-backup
    11  net0: name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.1,hwaddr=BC:24:11:17:FF:E8,ip=192.168.1.100/24,type=veth
    12  onboot: 1
    13  ostype: ubuntu
    14  parent: Docker_install_and_user_added
    15  rootfs: local-lvm:vm-401-disk-0,size=24G
    16  swap: 1024
    17  tags: docker
    18  unprivileged: 1
    19
    20  [Docker_install_and_user_added]
    21  #created a new user and added docker app. no docker file is added as of now
    22  arch: amd64
    23  cores: 4
    24  features: fuse=1,keyctl=1,nesting=1
    25  hostname: DockerHost
    26  memory: 2048
    27  net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=BC:24:11:17:FF:E8,ip=dhcp,ip6=dhcp,type=veth
    28  onboot: 1
    29  ostype: ubuntu
    30  rootfs: local-lvm:vm-401-disk-0,size=24G
    31  snaptime: 1778779417
    32  swap: 1024
    33  unprivileged: 1
    34  lxc.cgroup2.devices.allow: c 10:200 rwm
    35  lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file

Sun Jul 26, 02:13:23 PM "/"|root@pve:# fresh /etc/pve/lxc/401.conf

Sun Jul 26, 02:22:19 PM "/"|root@pve:# cat -n /etc/pve/lxc/401.conf
     1  arch: amd64
     2  cores: 4
     3  features: fuse=1,keyctl=1,nesting=1
     4  hostname: DockerHost
     5  memory: 2048
     6  mp0: /mnt/main-backup/homelab_backups/vikunja_lxc_backup,mp=/vikunja_backup
     7  mp1: /mnt/main-backup/homelab/apps,mp=/apps
     8  mp2: /mnt/main-backup,mp=/mnt/hdd-main-backup
     9  mp3: /mnt/entertainment/Games Setup,mp=/mnt/hdd-games
    10  mp4: /mnt/entertainment/Media,mp=/mnt/hdd-media
    11  net0: name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.1,hwaddr=BC:24:11:17:FF:E8,ip=192.168.1.100/24,type=veth
    12  onboot: 1
    13  ostype: ubuntu
    14  parent: Docker_install_and_user_added
    15  rootfs: local-lvm:vm-401-disk-0,size=24G
    16  swap: 1024
    17  tags: docker
    18  unprivileged: 1
    19
    20  [Docker_install_and_user_added]
    21  #created a new user and added docker app. no docker file is added as of now
    22  arch: amd64
    23  cores: 4
    24  features: fuse=1,keyctl=1,nesting=1
    25  hostname: DockerHost
    26  memory: 2048
    27  net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=BC:24:11:17:FF:E8,ip=dhcp,ip6=dhcp,type=veth
    28  onboot: 1
    29  ostype: ubuntu
    30  rootfs: local-lvm:vm-401-disk-0,size=24G
    31  snaptime: 1778779417
    32  swap: 1024
    33  unprivileged: 1
    34  lxc.cgroup2.devices.allow: c 10:200 rwm
    35  lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file

Sun Jul 26, 02:22:22 PM "/"|root@pve:# 
```

Now, we can try to switch on the LXCs and check if the LXC runs without errors in the new configuration.

```sh
Sun Jul 26, 02:25:17 PM "/"|root@pve:# pct start 400

Sun Jul 26, 02:25:34 PM "/"|root@pve:# 
Sun Jul 26, 02:25:38 PM "/"|root@pve:# pct start 401

Sun Jul 26, 02:25:48 PM "/"|root@pve:# 
```

So now our LXCs will use the new HDD without using the old ones.

### Updating settings
We now, check by logging into portainer and Samba
#### Portainer
Open `https://proxmox.tglservice.top/` in a browser, the result is a "This site can’t be reached" message.
Open `https://100.77.8.142:9443` the portainer is available to log in
Check containers to see what containers are not working: `https://100.77.8.142:9443/#!/3/docker/containers`
![084 containers after HDD change.png](/img/user/All%20Published%20Notes/Homelab/Images/084%20containers%20after%20HDD%20change.png)

many services are not started, we will validate why by looking at the stacks: `https://100.77.8.142:9443/#!/3/docker/stacks`

Open `docsify_mapper` which creates the container `docsify_mapper`. The setup looks good.

**The solution is to simply re-create the containers** as the drive mount point changed or the underlying format has changed.

### Conclusion
#### Key Takeaway for Your Homelab
Whenever you change Proxmox mount points (`mpX`), `fstab` entries, or underlying hard drives:
- Running `docker restart <container>` will often fail for containers with volume mounts because of cached inode handles.
- You should always **re-create** your Docker stacks so Docker re-initializes the volume bindings against the active mounts

#### Cleanup
We can also delete unnecessary files created by proxmox from the media and games folders like these:
- dump
- images
- import
- private
- snippets
- template





---

[^1]: 
[^2]: 

