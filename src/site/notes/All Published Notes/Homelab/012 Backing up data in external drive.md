---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/012-backing-up-data-in-external-drive/"}
---

created: 2026-05-19
updated: 2026-06-13

Given that I have external drives and I want my data stored in external drives as a backup in case my main hard-disk collapses, I will make a folder to backup only my data files for vikunja that we made earlier[^1].

### Creating a folder to hold backup

First, we need a folder in which all backups will be moved to, this folder should exist in the LXC as well as be mapped to a main folder inside my hard-drive. But to do this, we need the LXC to be in 'stopped' state or in simpler words, the *LXC should be shut down*.

I want to use '/dev/sdb1' drive, which is mounted as '/mnt/main-backup' in my machine for my backups.

```sh
root@pve:~# df -h
Filesystem            Size  Used Avail Use% Mounted on
udev                  7.5G     0  7.5G   0% /dev
tmpfs                 1.6G  2.0M  1.6G   1% /run
/dev/mapper/pve-root   68G  6.1G   59G  10% /
tmpfs                 7.6G   46M  7.5G   1% /dev/shm
efivarfs              128K   52K   72K  43% /sys/firmware/efi/efivars
tmpfs                 5.0M     0  5.0M   0% /run/lock
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
/dev/sda2            1022M  8.8M 1014M   1% /boot/efi
tmpfs                 7.6G     0  7.6G   0% /tmp
/dev/sdc1             932G  797G  135G  86% /mnt/games-setup
/dev/sdd1             932G  532G  400G  58% /mnt/media-hhd
/dev/sdb1             3.7T  220G  3.5T   6% /mnt/main-backup
/dev/fuse             128M   16K  128M   1% /etc/pve
tmpfs                 1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                 1.6G  4.0K  1.6G   1% /run/user/0
root@pve:~# ls /mnt/main-backup/
'$RECYCLE.BIN'   dump        images   Personal   Professional   System                       template
 Documents       Education   import   private    snippets      'System Volume Information'
root@pve:~# mkdir -p /mnt/main-backup/homelab_backups/vikunja_lxc_backup
root@pve:~# ls /mnt/main-backup/
'$RECYCLE.BIN'   dump        homelab_backups   import     private        snippets  'System Volume Information'
 Documents       Education   images            Personal   Professional   System     template
root@pve:~# chmod 777 /mnt/main-backup/homelab_backups/vikunja_lxc_backup
root@pve:~# # This maps the host backup folder to '/backup' inside LXC 401
root@pve:~# pct set 401 -mp0 /mnt/main-backup/homelab_backups/vikunja_lxc_backup,mp=/backup
root@pve:~# 
```

> [!Warning]
> `mp0` is for 0th mounting point, if it is in use, we can change to `mp1` or any other which is unused. It all works the same. To see all mount points in use, we can check it in proxmox in config of each pct.

So now, my Ubuntu LXC has 2 folders at the root level 
- `/backup`: to store backup which is a link to store in main host's '/mnt/main-backup' folder decided by the proxmox host
- `/opt/vikunja/`: which holds both db and data files needed for my vikunja application.

![037 vid 401 root files.png](/img/user/All%20Published%20Notes/Homelab/Images/037%20vid%20401%20root%20files.png)

Now we can use `/backup` for creating and storing the backups

### Creating a backup script
As we can easily copy the data between the two folders (we have 777 permission) manually, it might seem like a simple command to copy the files. But, in spirit of automation, we want to create a script that will create the backup, and we can plan the scheduling later.

The script will be stored as `/opt/backup-vikunja.sh` along side `/opt/vikunja/`, this is to avoid duplicating the script multiple times inside each backup by placing it in `/opt/vikunja/`.

`/opt/backup-vikunja.sh`:
```sh
#!/bin/bash
# version 1.0

# Define where to save the backup and timestamp
BACKUP_DIR="/backup"
TIMESTAMP=$(date +"%Y-%m-%d_%H-%m-%M")
BACKUP_NAME="vikunja_data_${TIMESTAMP}.tar.gz"

# Compress both folders into one archive directly onto the backup drive
tar -czf ${BACKUP_DIR}/${BACKUP_NAME} -C / opt/vikunja/files opt/vikunja/db

# Optional: Delete backups older than 30 days to save space
find ${BACKUP_DIR} -name "vikunja_data_*.tar.gz" -type f -mtime +30 -delete
```

make this script executable:
```sh
root@DockerHost:~# chmod +x /opt/backup-vikunja.sh
```

now we can run it by
```sh
root@DockerHost:~# /opt/backup-vikunja.sh
```

log:
```sh
root@DockerHost:~# ls -al /backup/
total 132
drwxrwxrwx  2 nobody nogroup 131072 May 19 16:34 .
drwxr-xr-x 22 root   root      4096 May 19 16:35 ..
root@DockerHost:~# /opt/backup-vikunja.sh
bash: /opt/backup-vikunja.sh: No such file or directory
root@DockerHost:~# fresh /opt/backup-vikunja.sh

A new version of fresh is available: 0.3.5 -> 0.3.7
Download from: https://github.com/sinelaw/fresh/releases/tag/v0.3.7

root@DockerHost:~# /opt/backup-vikunja.sh
bash: /opt/backup-vikunja.sh: Permission denied
root@DockerHost:~# chmod +x /opt/backup-vikunja.sh
root@DockerHost:~# /opt/backup-vikunja.sh
root@DockerHost:~# ls -al /backup/
total 7940
drwxrwxrwx  2 nobody nogroup  131072 May 19 16:55 .
drwxr-xr-x 22 root   root       4096 May 19 16:35 ..
-rwxrwxrwx  1 nobody nogroup 7898063 May 19 16:55 vikunja_data_2026-05-19_16-05-55.tar.gz
root@DockerHost:~# 
```

### Scheduling the backup to run automatically
I am lazy, so I will definitely stop taking manual backups after 1 week if I have to do it regularly. So a better option is to automate it, but I do not want to take backups when no data has changed, or minimal data has changed.
A idea would be to try to take a backup everytime I am re-booting the LXC, and to compare the change in folder size, if the size difference is below 2KB (2000 characters worth in vikunja), I will skip the backup.
Having a fixed time is best only if the homelab will be 'on' 24x7.
I also may make very little change as I will be using it alone, so I can add a bypass to file check if it has been 7 days since I last took a backup.

As the backup needs to check the previous backup exists or not, we need to change the script we want to use:
```sh
#!/bin/bash
# version 2.0

BACKUP_DIR="/backup"
TIMESTAMP=$(date +"%Y-%m-%d_%H-%m-%M")
NEW_BACKUP="${BACKUP_DIR}/vikunja_data_${TIMESTAMP}.tar.gz"
THRESHOLD_BYTES=2048 # 2KB size threshold
MAX_DAYS_WITHOUT_BACKUP=7

# 1. Calculate total current size of live data folders in bytes
CURRENT_SIZE=$(du -sb /opt/vikunja/files /opt/vikunja/db | awk '{sum+=$1} END {print sum}')

# 2. Find the most recent backup file in the backup directory
LAST_BACKUP=$(ls -t ${BACKUP_DIR}/vikunja_data_*.tar.gz 2>/dev/null | head -n 1)

# 3. Evaluation logic if a previous backup exists
if [ -f "$LAST_BACKUP" ]; then
    
    # Check A: Has it been more than 7 days since the last backup?
    LAST_BACKUP_TIME=$(stat -c %Y "$LAST_BACKUP")
    CURRENT_TIME=$(date +%s)
    AGE_DAYS=$(( (CURRENT_TIME - LAST_BACKUP_TIME) / 86400 ))
    
    if [ $AGE_DAYS -ge $MAX_DAYS_WITHOUT_BACKUP ]; then
        echo "Safety net triggered: It has been ${AGE_DAYS} days since the last backup. Forcing backup..."
    else
        # Check B: Size threshold comparison (only evaluated if the backup is under 7 days old)
        LAST_SIZE=$(gzip -l "$LAST_BACKUP" | tail -n 1 | awk '{print $2}')
        SIZE_DIFF=$((CURRENT_SIZE - LAST_SIZE))
        
        # Treat negative differences (deleted data) as absolute changes
        if [ $SIZE_DIFF -lt 0 ]; then
            SIZE_DIFF=$(( -SIZE_DIFF ))
    	fi
    	
        if [ $SIZE_DIFF -lt $THRESHOLD_BYTES ]; then
            echo "Data change (${SIZE_DIFF} bytes) is below 2KB and last backup is only ${AGE_DAYS} days old. Skipping."
            exit 0
        fi
        echo "Significant data changes (${SIZE_DIFF} bytes) detected. Archiving..."
    fi
else
    echo "No previous backup found. Running initial backup..."
fi

# 4. Execute the backup archive
tar -czf ${NEW_BACKUP} -C / opt/vikunja/files opt/vikunja/db
echo "Backup successfully created: ${NEW_BACKUP}"

# 5. Keep the house clean: Keep only the last 30 distinct backups
ls -t ${BACKUP_DIR}/vikunja_data_*.tar.gz 2>/dev/null | tail -n +31 | xargs -r rm

```

Now, we can update the data in `crontab -e` to auto run this script on reboot of the LXC:
terminal:
```sh
root@DockerHost:~# crontab -e
no crontab for root - using an empty one
crontab: installing new crontab
root@DockerHost:~# # after saving the file changes, we will return here
root@DockerHost:~# 
```

In the crontab opened, add this line:
```
@reboot /opt/backup-vikunja.sh > /dev/null 2>&1
```

This ensures the task runs on every LXC re-boot. This way before docker or portainer can start, we will take backups.


Additional:
> [!Note]
> If we are mounting lot of paths and need to check which LXC is mounted to which folder, we can use `pct config 401 | grep mp` to get data about all pct configurations that have mount points.

```sh
root@pve:~# pct config 401 | grep mp
mp0: /mnt/main-backup/homelab_backups/vikunja_lxc_backup,mp=/vikunja_backup
mp1: /mnt/main-backup/homelab/apps,mp=/apps
root@pve:~#
```



---

[^1]: [[All Published Notes/Homelab/010 Self hosted Task Manager - Vikunja\|010 Self hosted Task Manager - Vikunja]]
[^2]: 