---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/formatting-new-hard-drive/"}
---

created: 2026-07-23
updated: 2026-07-23

I bought a new hard drive of 2TB, formatting it to `ext4` format and preparing it to use with my Linux machines:
```sh
Thu Jul 23, 10:26:30 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           6.2G  2.6M  6.2G   1% /run
efivarfs        128K   60K   64K  49% /sys/firmware/efi/efivars
/dev/sda2       938G  524G  367G  59% /
tmpfs            31G  154M   31G   1% /dev/shm
tmpfs           5.0M   20K  5.0M   1% /run/lock
/dev/nvme1n1p1   96M   38M   59M  40% /boot/efi
tmpfs           6.2G  236K  6.2G   1% /run/user/1000
/dev/sdc2       1.9T  159M  1.9T   1% /media/thegamelearner/ADATA HD710M PRO

Thu Jul 23, 10:46:21 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$

Thu Jul 23, 10:49:15 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ sudo umount /dev/sdc2
[sudo] password for thegamelearner:           

Thu Jul 23, 10:49:21 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ 

Thu Jul 23, 10:51:58 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ sudo mkfs.ext4 -L "Rish Main Backup" /dev/sdc2
mke2fs 1.47.0 (5-Feb-2023)
/dev/sdc2 contains a ext4 file system labelled 'ADATA HD710M PRO'
        created on Thu Jul 23 22:50:51 2026
Proceed anyway? (y,N) y
Creating filesystem with 488374272 4k blocks and 122332032 inodes
Filesystem UUID: e0c294c8-ec80-45bb-8e19-13ef537bf24d
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
        102400000, 214990848

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done       


Thu Jul 23, 10:52:14 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ 

Thu Jul 23, 10:52:14 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           6.2G  2.6M  6.2G   1% /run
efivarfs        128K   60K   64K  49% /sys/firmware/efi/efivars
/dev/sda2       938G  524G  367G  59% /
tmpfs            31G  155M   31G   1% /dev/shm
tmpfs           5.0M   20K  5.0M   1% /run/lock
/dev/nvme1n1p1   96M   38M   59M  40% /boot/efi
tmpfs           6.2G  236K  6.2G   1% /run/user/1000

Thu Jul 23, 10:56:19 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ sudo mkdir -p "/media/thegamelearner/Rish Main Backup"

Thu Jul 23, 10:56:21 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ sudo mount /dev/sdc2 "/media/thegamelearner/Rish Main Backup"

Thu Jul 23, 10:56:32 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           6.2G  2.6M  6.2G   1% /run
efivarfs        128K   60K   64K  49% /sys/firmware/efi/efivars
/dev/sda2       938G  524G  367G  59% /
tmpfs            31G  158M   31G   1% /dev/shm
tmpfs           5.0M   20K  5.0M   1% /run/lock
/dev/nvme1n1p1   96M   38M   59M  40% /boot/efi
tmpfs           6.2G  236K  6.2G   1% /run/user/1000
/dev/sdc2       1.8T   28K  1.7T   1% /media/thegamelearner/Rish Main Backup

Thu Jul 23, 10:56:34 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ 

Thu Jul 23, 10:56:34 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ sudo chown -R $USER:$USER "/media/thegamelearner/Rish Main Backup"

Thu Jul 23, 10:58:23 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ ls -al /media/thegamelearner/
total 16
drwxr-x---+ 4 root           root           4096 Jul 23 22:56  .
drwxr-xr-x  5 root           root           4096 Apr 25 22:01  ..
drwxr-xr-x  2 root           root           4096 Feb 15 23:03 'New Volume'
drwxr-xr-x  3 thegamelearner thegamelearner 4096 Jul 23 22:52 'Rish Main Backup'

Thu Jul 23, 10:58:27 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/FSM"|
thegamelearner@thegamelearner-MS-7E12:$ 
```

Now, the drive can be used as ext4 format drive to hold data.



---

[^1]: 
[^2]: 

