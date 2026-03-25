---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/002-first-setup-in-proxmox/"}
---

created: 2026-03-21
updated: 2026-03-22

I am new at this moment, so I am following lot of guide and youtube.

## Setup
#### Change the repositories used to upgrade Proxmox
Open the proxmox connection and navigate to the main system (in my case pve). 
Change APT repositories from enterprise to No-Subscription:
![001 Proxmox No Subscription.png](/img/user/All%20Published%20Notes/Homelab/Images/001%20Proxmox%20No%20Subscription.png)

Select the two enterprise subscription and disable them and add a repository called "No-Subscription". you should get a result similar to the image above.

Click on Updates and refresh, this will refresh all repositories and check for updates:
![002 Proxmox refresh.png](/img/user/All%20Published%20Notes/Homelab/Images/002%20Proxmox%20refresh.png)

After "Task OK" we can use the refreshed repositories for getting the available updates. Click on Update beside the refresh button, this should open a new terminal with yes no option to upgrade:
![003 Proxmox upgrade.png](/img/user/All%20Published%20Notes/Homelab/Images/003%20Proxmox%20upgrade.png)

Once upgrade is successful, it is better to reboot the system.

#### Define your storage
In my case, I have a single SSD in my machine which has 256 GB of space including the OS. These are divided into 2 sections ('local' and 'local-lvm') which has 72.72 GB and 151.64 GB, for the initial use case, this works for me. If you want to change it, it would be better to use 'LVM-Thin' as it only uses the bare minimum memory that you need.

#### Change system time
In my case, the time shown in logs does not match my local time, this might be due to some error so i want to understand and fix it.
I checked the shell time and setting using `timedatectl status` which shows local setup, this is correct but the logs still shows time in UTC (not error, just different format). On deeper dive, my browser is LibreWolf which uses UTC time, so logs are coming according to it's settings.
![004 proxmox time data.png](/img/user/All%20Published%20Notes/Homelab/Images/004%20proxmox%20time%20data.png)

#### Change default text editor
I do not like nano, but it is the only text editor we have if we need to change files. I will change my default editor to [fresh](https://getfresh.dev/) editor.
##### Micro
According to official documentation, to install, download `.deb` file and then run install using 'dpkg'. As I do not know enough about proxmox to have a URL and download and find this file, I will go with [micro](https://micro-editor.github.io/).

```sh
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@pve:~# which curl
/usr/bin/curl
root@pve:~# curl https://getmic.ro | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  9728  100  9728    0     0   7246      0  0:00:01  0:00:01 --:--:--  7248
Detected platform: linux64
Latest Version: 2.0.15
Downloading https://github.com/micro-editor/micro/releases/download/v2.0.15/micro-2.0.15-linux64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 4969k  100 4969k    0     0  2980k      0  0:00:01  0:00:01 --:--:-- 11.2M
micro-2.0.15/micro
/=====================================\\
|   update-alternatives is supported   |
\\=====================================/

getmicro can use update-alternatives to register micro as a system text editor.
For example, this will allow `crontab -e` open the cron file with micro.

To enable this feature, define the GETMICRO_REGISTER variable or use the URL
`https://getmic.ro/r`.

Note that you must install micro to a directory accessible to all users when doing
this, typically /usr/bin. cd to that directory before running this script.

E.g.:

  $ cd /usr/bin
  $ curl https://getmic.ro/r | sudo sh

or

  $ su - root -c "cd /usr/bin; wget -O- https://getmic.ro | GETMICRO_REGISTER=y sh"



 __  __ _                  ___           _        _ _          _ _
|  \/  (_) ___ _ __ ___   |_ _|_ __  ___| |_ __  | | | ___  __| | |
| |\/| | |/ __| '__/ _ \   | || '_ \/ __| __/ _\ | | |/ _ \/ _  | |
| |  | | | (__| | | (_) |  | || | | \__ \ || (_| | | |  __/ (_| |_|
|_|  |_|_|\___|_|  \___/  |___|_| |_|___/\__\__,_|_|_|\___|\__,_(_)

Micro has been downloaded to the current directory.
You can run it with:

./micro

root@pve:~# mv micro /usr/local/bin/
root@pve:~# micro -version
Version: 2.0.15
Commit hash: 6a62575b
Compiled on December 31, 2025
root@pve:~# 
```

Micro is a bad choice for shell, as it closes file with 'ctrl+q' but most browsers on windows will quit the app with same shortcut.
uninstalling micro:
```sh
root@pve:~# apt purge micro
Package 'micro' is not installed, so not removed
Summary:
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
root@pve:~# which micro
/usr/local/bin/micro
root@pve:~# apt purge micro
Package 'micro' is not installed, so not removed
Summary:
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
root@pve:~# # this is because micro was installed using curl
root@pve:~# rm /usr/local/bin/micro
root@pve:~# rm -rf ~/.config/micro
root@pve:~# which micro
root@pve:~# 
```
##### Midnight Commander
installed midnight commander for time being:
```sh
root@pve:~# apt-get install mc
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libgpm2 mailcap mc-data unzip
Suggested packages:
  gpm antiword arc | arcanist arj cabextract catdoc texlive-binaries clzip | lunzip | lzd | lzip | lziprecover | minilzip | pdlzip | plzip ctorrent dbview
  default-jdk-headless djview4 djvulibre-bin elinks epub-utils | ncbi-entrez-direct exif gettext ghostscript glade gputils gv imagemagick info jlha-utils | lhasa
  kchmviewer libaspell-dev libbatik-java libchm-bin liblz4-tool libxml2-utils links2 links | w3m | lynx lyx mikmod mpg321 mplayer odt2txt | unoconv p7zip-full par2
  poedit | potool poppler-utils procyon-decompiler python3-boto python3-tz rar rpm sox timidity unace | unace-nonfree unalz unar unrar | unrar-free vorbis-tools
  wimtools wv xpdf | pdf-viewer xpmutils zip
The following NEW packages will be installed:
  libgpm2 mailcap mc mc-data unzip
0 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
Need to get 2,111 kB of archives.
After this operation, 8,587 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://deb.debian.org/debian trixie/main amd64 libgpm2 amd64 1.20.7-11+b2 [14.4 kB]
Get:2 http://deb.debian.org/debian trixie/main amd64 mailcap all 3.74 [32.8 kB]
Get:3 http://deb.debian.org/debian trixie/main amd64 mc-data all 3:4.8.33-1+deb13u1 [1,341 kB]
Get:4 http://deb.debian.org/debian trixie/main amd64 mc amd64 3:4.8.33-1+deb13u1 [550 kB]
Get:5 http://deb.debian.org/debian trixie/main amd64 unzip amd64 6.0-29 [173 kB]
Fetched 2,111 kB in 3s (835 kB/s)
Selecting previously unselected package libgpm2:amd64.
(Reading database ... 60036 files and directories currently installed.)
Preparing to unpack .../libgpm2_1.20.7-11+b2_amd64.deb ...
Unpacking libgpm2:amd64 (1.20.7-11+b2) ...
Selecting previously unselected package mailcap.
Preparing to unpack .../archives/mailcap_3.74_all.deb ...
Unpacking mailcap (3.74) ...
Selecting previously unselected package mc-data.
Preparing to unpack .../mc-data_3%3a4.8.33-1+deb13u1_all.deb ...
Unpacking mc-data (3:4.8.33-1+deb13u1) ...
Selecting previously unselected package mc.
Preparing to unpack .../mc_3%3a4.8.33-1+deb13u1_amd64.deb ...
Unpacking mc (3:4.8.33-1+deb13u1) ...
Selecting previously unselected package unzip.
Preparing to unpack .../unzip_6.0-29_amd64.deb ...
Unpacking unzip (6.0-29) ...
Setting up libgpm2:amd64 (1.20.7-11+b2) ...
Setting up unzip (6.0-29) ...
Setting up mc-data (3:4.8.33-1+deb13u1) ...
Setting up mailcap (3.74) ...
Setting up mc (3:4.8.33-1+deb13u1) ...
update-alternatives: using /usr/bin/mcview to provide /usr/bin/view (view) in auto mode
Processing triggers for man-db (2.13.1-1) ...
Processing triggers for libc-bin (2.41-12+deb13u2) ...
```
I found a way to install fresh text editor, so removing mc:
```sh
root@pve:~# apt ourge mc
Error: Invalid operation ourge
root@pve:~# apt purge mc
The following packages were automatically installed and are no longer required:
  mailcap  mc-data  unzip
Use 'apt autoremove' to remove them.

REMOVING:
  mc*

Summary:
  Upgrading: 0, Installing: 0, Removing: 1, Not Upgrading: 0
  Freed space: 1,628 kB

Continue? [Y/n] y
(Reading database ... 60582 files and directories currently installed.)
Removing mc (3:4.8.33-1+deb13u1) ...
update-alternatives: using /usr/bin/vim.tiny to provide /usr/bin/view (view) in auto mode
Processing triggers for mailcap (3.74) ...
(Reading database ... 60490 files and directories currently installed.)
Purging configuration files for mc (3:4.8.33-1+deb13u1) ...
root@pve:~# 
```
##### Fresh
To install Fresh run the following command in terminal
```sh
curl https://raw.githubusercontent.com/sinelaw/fresh/refs/heads/master/scripts/install.sh | sh
```
which will identify the machine, download the necessary file and install it.
Now, we can change alternatives to ensure by default we will use fresh in terminal:
```sh
root@pve:~# which editor
/usr/bin/editor
root@pve:~# ls -a /usr/bin/editor
/usr/bin/editor
root@pve:~# ls -al /usr/bin/editor
lrwxrwxrwx 1 root root 24 Apr  6  2025 /usr/bin/editor -> /etc/alternatives/editor
root@pve:~# ls -al /etc/alternatives/editor
lrwxrwxrwx 1 root root 9 Apr  6  2025 /etc/alternatives/editor -> /bin/nano
root@pve:~# which fresh
/usr/bin/fresh
root@pve:~# update-alternatives --config editor
There are 2 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path               Priority   Status
------------------------------------------------------------
* 0            /bin/nano           40        auto mode
  1            /bin/nano           40        manual mode
  2            /usr/bin/vim.tiny   15        manual mode

Press <enter> to keep the current choice[*], or type selection number: 
root@pve:~# update-alternatives --install /usr/bin/editor editor /usr/bin/fresh 50
update-alternatives: using /usr/bin/fresh to provide /usr/bin/editor (editor) in auto mode
root@pve:~# update-alternatives --config editor
There are 3 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path               Priority   Status
------------------------------------------------------------
* 0            /usr/bin/fresh      50        auto mode
  1            /bin/nano           40        manual mode
  2            /usr/bin/fresh      50        manual mode
  3            /usr/bin/vim.tiny   15        manual mode

Press <enter> to keep the current choice[*], or type selection number: 
root@pve:~# ls -l /etc/alternatives/editor
lrwxrwxrwx 1 root root 14 Mar 22 17:19 /etc/alternatives/editor -> /usr/bin/fresh
root@pve:~# 
```

#### My Hard disks are not showing up
I now proceeded to attach my 3 hard disks(1, 2, 4 TB) which I want to use as NAS. The hard disks do not show up anywhere.

Again in shell, i will search all disks attached:
```sh
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@pve:~# lsblk
NAME               MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                  8:0    0 238.5G  0 disk 
├─sda1               8:1    0  1007K  0 part 
├─sda2               8:2    0     1G  0 part /boot/efi
└─sda3               8:3    0 237.5G  0 part 
  ├─pve-swap       252:0    0     8G  0 lvm  [SWAP]
  ├─pve-root       252:1    0  69.4G  0 lvm  /
  ├─pve-data_tmeta 252:2    0   1.4G  0 lvm  
  │ └─pve-data     252:4    0 141.2G  0 lvm  
  └─pve-data_tdata 252:3    0 141.2G  0 lvm  
    └─pve-data     252:4    0 141.2G  0 lvm  
sdb                  8:16   0 931.5G  0 disk 
└─sdb1               8:17   0 931.5G  0 part 
sdc                  8:32   0 931.5G  0 disk 
└─sdc1               8:33   0 931.5G  0 part 
sdd                  8:48   0   3.6T  0 disk 
└─sdd1               8:49   0   3.6T  0 part 
root@pve:~# 
```

Sometimes Proxmox doesn't immediately see newly connected drives. Run this command to force a rescan of all storage devices without rebooting
```sh
root@pve:~# qm disk rescan
rescan volumes...
root@pve:~# 
```

It did not work, so now I will first try to find the details and then map a mount point to access the directories, to do this, I need to know the harddrive in each name (sdb, sdc, sdd), so I will use `fdisk -l`:
```sh
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@pve:~# lsblk
NAME               MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                  8:0    0 238.5G  0 disk 
├─sda1               8:1    0  1007K  0 part 
├─sda2               8:2    0     1G  0 part /boot/efi
└─sda3               8:3    0 237.5G  0 part 
  ├─pve-swap       252:0    0     8G  0 lvm  [SWAP]
  ├─pve-root       252:1    0  69.4G  0 lvm  /
  ├─pve-data_tmeta 252:2    0   1.4G  0 lvm  
  │ └─pve-data     252:4    0 141.2G  0 lvm  
  └─pve-data_tdata 252:3    0 141.2G  0 lvm  
    └─pve-data     252:4    0 141.2G  0 lvm  
sdb                  8:16   0 931.5G  0 disk 
└─sdb1               8:17   0 931.5G  0 part 
sdc                  8:32   0 931.5G  0 disk 
└─sdc1               8:33   0 931.5G  0 part 
sdd                  8:48   0   3.6T  0 disk 
└─sdd1               8:49   0   3.6T  0 part 
root@pve:~# fdisk -l
Disk /dev/sda: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: EVM25/256GB     
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 086A3F53-9EB5-4D94-A196-B1D8DC9FF9D5

Device       Start       End   Sectors   Size Type
/dev/sda1       34      2047      2014  1007K BIOS boot
/dev/sda2     2048   2099199   2097152     1G EFI System
/dev/sda3  2099200 500118158 498018959 237.5G Linux LVM


Disk /dev/mapper/pve-swap: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/pve-root: 69.37 GiB, 74482450432 bytes, 145473536 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 931.48 GiB, 1000170586112 bytes, 1953458176 sectors
Disk model: My Passport 259F
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 3390577B-F358-48AC-8CF0-2485ECCBF2AA

Device     Start        End    Sectors   Size Type
/dev/sdb1   2048 1953456127 1953454080 931.5G Microsoft basic data


Disk /dev/sdc: 931.51 GiB, 1000204885504 bytes, 1953525167 sectors
Disk model: Backup+ BK      
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x935662a0

Device     Boot Start        End    Sectors   Size Id Type
/dev/sdc1  *     2048 1953521663 1953519616 931.5G  7 HPFS/NTFS/exFAT


Disk /dev/sdd: 3.64 TiB, 4000752599040 bytes, 7813969920 sectors
Disk model: Elements 2621   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 5E7F0D19-8DF6-4399-91D0-DD87D521AE6E

Device     Start        End    Sectors  Size Type
/dev/sdd1   2048 7813967871 7813965824  3.6T Microsoft basic data
root@pve:~# 
```
now, I know which hard drive has what name and what type of data, but for documentation:
`/dev/sdb`: media harddrive (Movies, TV shows)
`/dev/sdc`: Games Setup files (Games Setup)
`/dev/sdd`: Important data, all bills and personal data (Main Backup)

These drives need a mount point to be found in files, so we will have to create mount points for the disks.
```sh
mkdir -p /mnt/media-hhd
mkdir -p /mnt/games-setup
mkdir -p /mnt/main-backup
```

So we will add each drive with mount point:
```sh
mount -t ntfs-3g /dev/sdb1 /mnt/media-hhd
mount -t ntfs-3g /dev/sdc1 /mnt/games-setup
mount -t ntfs-3g /dev/sdd1 /mnt/main-backup
```

If you get an error about `ntfs-3g` not installed, install it:
```sh
apt update && apt install ntfs-3g -y
```

We will have to add this in `/etc/fstab` to retain it across re-boot:
```sh
echo "/dev/sdb1 /mnt/passport1tb ntfs-3g defaults 0 0" >> /etc/fstab
```

Improvement for more in-depth control:
If we mount using the current sdb,sdd, etc, these may change if we ever change the ports or remove and re-attach a storage, so we also want to find their UUIDs. As all my disks have no partitioning, I can use `sdb1` but if you have partitions, get the UUIDs for all partitions.
```sh
root@pve:~# blkid /dev/sdb1 /dev/sdc1 /dev/sdd1
/dev/sdb1: LABEL="Movies TV" UUID="797F-72BF" BLOCK_SIZE="512" TYPE="exfat" PARTLABEL="My Passport" PARTUUID="2c14786a-4da1-4221-a63e-f7a4cd24fcdc"
/dev/sdc1: LABEL="Games Setup" UUID="EBDB-185F" BLOCK_SIZE="512" TYPE="exfat" PARTUUID="935662a0-01"
/dev/sdd1: LABEL="4TB Main Us" UUID="ED5E-0B40" BLOCK_SIZE="512" TYPE="exfat" PARTLABEL="primary" PARTUUID="8bd2f2a4-a886-44ec-aebb-25fe2f41fa93"
root@pve:~# 
```
Making changes in `/etc/fstab` will allow us to define them as fixed drives based on UUIDs, but it may not be a great idea to manually change the data regularly as your disks change. If you are planning on adding a drive as NAS or leave the hard disk and the proxmox running for long, only then this is a good idea.
use micro or nano to update `/etc/fstab` file. At the bottom, add UUID with the mount point, add safety to avoid any issues in not finding the disks:
```
# Media Harddrive (sdb1)
UUID="797F-72BF" /mnt/media-hhd auto nosuid,nodev,nofail 0 2

# Games Setup (sdc1)
UUID="EBDB-185F" /mnt/games-setup auto nosuid,nodev,nofail 0 2

# Main Backup (sdd1)
UUID="ED5E-0B40" /mnt/main-backup auto nosuid,nodev,nofail 0 2
```

> [!Note] Using `auto` allows Linux to guess if it's NTFS or exFAT. `nofail` ensures that if a drive is unplugged, your computer still boots up.

Now, that the setup is done, we need to mount the actual drives with `mount -a`, this will mount the drives but to refresh the data, we need `systemctl daemon-reload` command.
```sh
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
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
root@pve:~#  blkid /dev/sdb1 /dev/sdc1 /dev/sdd1
/dev/sdb1: LABEL="Movies TV" UUID="797F-72BF" BLOCK_SIZE="512" TYPE="exfat" PARTLABEL="My Passport" PARTUUID="2c14786a-4da1-4221-a63e-f7a4cd24fcdc"
/dev/sdc1: LABEL="Games Setup" UUID="EBDB-185F" BLOCK_SIZE="512" TYPE="exfat" PARTUUID="935662a0-01"
/dev/sdd1: LABEL="4TB Main Us" UUID="ED5E-0B40" BLOCK_SIZE="512" TYPE="exfat" PARTLABEL="primary" PARTUUID="8bd2f2a4-a886-44ec-aebb-25fe2f41fa93"
root@pve:~# mkdir -p /mnt/media-hhd
root@pve:~# ls /mnt/games-setup
ls: cannot access '/mnt/games-setup': No such file or directory
root@pve:~# mkdir -p /mnt/games-setup
root@pve:~# mkdir -p /mnt/main-backup
root@pve:~# ls /mnt/games-setup
root@pve:~# mount -t ntfs-3g /dev/sdb1 /mnt/media-hhd
mount: /mnt/media-hhd: unknown filesystem type 'ntfs-3g'.
       dmesg(1) may have more information after failed mount system call.
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
root@pve:~# systemctl daemon-reload
root@pve:~# ls /mnt/games-setup
root@pve:~# ls /mnt/
games-setup  main-backup  media-hhd
root@pve:~# ls /mnt/main-backup/
root@pve:~# ls -al /mnt/main-backup/
total 8
drwxr-xr-x 2 root root 4096 Mar 21 20:08 .
drwxr-xr-x 5 root root 4096 Mar 21 20:08 ..
root@pve:~# mount -a
root@pve:~# ls -al /mnt/main-backup/
total 1156
drwxr-xr-x 10 root root 131072 Mar 21 20:11  .
drwxr-xr-x  5 root root   4096 Mar 21 20:08  ..
drwxr-xr-x  2 root root 131072 Feb 12 18:29 '$RECYCLE.BIN'
drwxr-xr-x 10 root root 131072 Jan  4 12:40  Documents
drwxr-xr-x  3 root root 131072 Jan  3 22:59  Education
drwxr-xr-x  6 root root 131072 Jan  4 12:58  Personal
drwxr-xr-x 10 root root 131072 Jan  3 22:58  Professional
drwxr-xr-x  4 root root 131072 Jan  3 18:10  System
drwxr-xr-x  2 root root 131072 Jan 16 08:11 'System Volume Information'
drwxr-xr-x  4 root root 131072 Mar 19 12:38  .Trash-1000
root@pve:~# 
```
As seen above, we can access the files in our terminal using the mount points created only after `mount -a` command.
To confirm what drives are mounted at which path, we can use `df -h` command:
```sh
root@pve:~# df -h
Filesystem            Size  Used Avail Use% Mounted on
udev                  7.5G     0  7.5G   0% /dev
tmpfs                 1.6G  1.5M  1.6G   1% /run
/dev/mapper/pve-root   68G  5.4G   59G   9% /
tmpfs                 7.6G   46M  7.5G   1% /dev/shm
efivarfs              128K   52K   72K  43% /sys/firmware/efi/efivars
tmpfs                 5.0M     0  5.0M   0% /run/lock
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
/dev/sda2            1022M  8.8M 1014M   1% /boot/efi
tmpfs                 7.6G     0  7.6G   0% /tmp
/dev/sdd1             932G  797G  135G  86% /mnt/games-setup
/dev/sdb1             932G  532G  400G  58% /mnt/media-hhd
/dev/fuse             128M   16K  128M   1% /etc/pve
/dev/sdc1             3.7T  220G  3.5T   6% /mnt/main-backup
tmpfs                 1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                 1.6G  4.0K  1.6G   1% /run/user/0
root@pve:~# 
```

After all this, we still will not see the drives in our Proxmox GUI, as the drives need to be added as a data storage device to be visible in the 'Datacenter'.

Now, I need to map this in my proxmox, so, I will use GUI and follow these steps for each drive:
Add as storage in Proxmox GUI
- Go to **Datacenter → Storage → Add → Directory**
- **ID**: a name like `media`
- **Directory**: `/mnt/media-hhd`
- **Content**: select the types you want (e.g., ISO images, VZDump backup files – you can uncheck VM/Container disks if you don’t plan to run VMs from it) 
- Click **Add**
Doing this for all drives shows the directories as a available storage media.
The 'Content' section is not very relevant if you want to use the drives as NAS data pool, but it is useful to set it to check on summary and other data from the hard drive when needed.

#### Accessing proxmox shell with SSH
As I am going to run shell script multiple times, it is better to save and use a 'profile' in my terminal ('Tabby') rather than change to shell in proxmox, as it deletes old commands from the shell when we go to other screen and re-enter it.

Step 1: Try to ssh into proxmox without profile
we try to SSH using the port we see in web GUI, this fails for some reason, so we use `ssh root@192.168.31.101` and leave the port empty:
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ ssh root@192.168.31.101 -p 8006
Connection closed by 192.168.31.101 port 8006
thegamelearner@thegamelearner-MS-7E12 ~ $ ssh -p 8006 root@192.168.31.101
Connection closed by 192.168.31.101 port 8006
thegamelearner@thegamelearner-MS-7E12 ~ $ ssh root@192.168.31.101
The authenticity of host '192.168.31.101 (192.168.31.101)' can't be established.
ED25519 key fingerprint is SHA256:vSN4OHKq2F5pXkV73nGLnqfQmEo1TbNm6rOL1tMteI4.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '192.168.31.101' (ED25519) to the list of known hosts.
root@192.168.31.101's password: 
Linux pve 6.17.13-2-pve #1 SMP PREEMPT_DYNAMIC PMX 6.17.13-2 (2026-03-13T08:06Z) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@pve:~# exit
logout
Connection to 192.168.31.101 closed.
thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

Step 2: define a profile in Tabby to connect to same and save the password to avoid writing it multiple times:
We will open our Tabby and write the host name and leave port empty(empty port is 22 which is default choice for proxmox):
![005 proxmox SSH profile.png](/img/user/All%20Published%20Notes/Homelab/Images/005%20proxmox%20SSH%20profile.png)

Using this, when you run, it may ask you to enter and save password, do that and you should see a terminal screen like this: 
```sh
 SSH  Connecting to root@192.168.31.101
 SSH  Host key fingerprint:
 SSH ***************************************************
Linux pve 6.17.13-2-pve #1 SMP PREEMPT_DYNAMIC PMX 6.17.13-2 (2026-03-13T08:06Z) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Mar 22 16:40:44 2026 from 192.168.31.204
root@pve:~# 
```
Screen shot asking password, notice the remember check box is ticked:
![006 Proxmox SSH Enter password.png](/img/user/All%20Published%20Notes/Homelab/Images/006%20Proxmox%20SSH%20Enter%20password.png)
After correct password:
![007 Proxmox ssh Successfully connected.png](/img/user/All%20Published%20Notes/Homelab/Images/007%20Proxmox%20ssh%20Successfully%20connected.png)
Now, I can connect to proxmox shell without having to open shell in proxmox.
For consistency, I am installing fresh on my machine, same as proxmox.

#### Accessing my Hard disks as remote drives
to start
- suggestion by deepseek: "Create an unprivileged LXC container (e.g., Debian or Ubuntu), Mount the drives on the Proxmox host (e.g., under /mnt/diskX) and bind‑mount those directories into the container, Inside the container, install Samba/NFS and share the bind‑mounted folders"
	- Inside the same LXC, you can run FileBrowser. Since you wanted to experiment with **Docker**, this is where you'd install Docker and run the FileBrowser container pointing to those same `/mnt/media` folders.
This was done in next note [[All Published Notes/Homelab/003 Proxmox - HDD as NAS\|003 Proxmox - HDD as NAS]]







---

[^1]: https://www.youtube.com/watch?v=qmSizZUbCOA
[^2]: https://www.youtube.com/watch?v=fNyC9NG2qHw&t=2511s
[^3]: https://www.youtube.com/watch?v=5j0Zb6x_hOk
[^4]: https://www.youtube.com/watch?v=b1BztUYB7VI
[^5]: https://www.youtube.com/watch?v=Hu3t8pcq8O0
[^6]: https://www.youtube.com/watch?v=BqZipmXuPlc
[^7]: 
[^8]: 
[^10]: 





