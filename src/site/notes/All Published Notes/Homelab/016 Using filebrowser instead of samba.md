---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/016-using-filebrowser-instead-of-samba/"}
---

created: 2026-07-04
updated: 2026-07-05

As mentioned before, I have a samba server running in my LXC (400) which can access my hard disks and I can read from 2/3 as a guest and can also login to my LXC with credentials to write to all of them. This enforces the security on a user id level so this was a great starting point.

As it was a working solution, I have not edited it till now, but now, I would like to access it on a browser and use it as a service even if I am out of my home using my Netbird VPN.

For achieving this, I will be using filebrowser.
Here, we will change the security aspect from user based to browser based authorization on this container.

> [!Warning]
> File Browser is great if you do not want a guest access or if all your directories are grouped, so each user can be given access to one directory. 
> I skipped it as I need these features


### Understanding Current state *(Optional)*
I have updated my `/etc/fstab` to ensure each hard drive I have will connect to same mount drives:
```sh
root@pve:~# cat -n /etc/fstab
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
root@pve:~# 
```

using `df -h`, let's find the state of the hard drives connected to my host machine:
```sh
root@pve:~# df -h
Filesystem            Size  Used Avail Use% Mounted on
udev                  7.5G     0  7.5G   0% /dev
tmpfs                 1.6G  2.1M  1.6G   1% /run
/dev/mapper/pve-root   68G  8.5G   56G  14% /
tmpfs                 7.6G   40M  7.5G   1% /dev/shm
efivarfs              128K   52K   72K  43% /sys/firmware/efi/efivars
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
tmpfs                 5.0M     0  5.0M   0% /run/lock
tmpfs                 1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
tmpfs                 7.6G     0  7.6G   0% /tmp
/dev/sdd1             932G  797G  135G  86% /mnt/games-setup
/dev/sdb1             932G  532G  400G  58% /mnt/media-hhd
/dev/sda2            1022M  8.8M 1014M   1% /boot/efi
/dev/sdc1             3.7T  221G  3.5T   6% /mnt/main-backup
/dev/fuse             128M   20K  128M   1% /etc/pve
tmpfs                 1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs                 1.6G  4.0K  1.6G   1% /run/user/0
root@pve:~# 
```

The three Hard drives(sdb1, sdc1, sdd1) are mounted on `/mnt/` directories with 'NAS_User' as the user with control and 'NAS_User_Group' as the group with all controls:
```sh
root@pve:~# ls -al /mnt/
total 396
drwxr-xr-x  6 root     root             4096 May 16 17:59 .
drwxr-xr-x 18 root     root             4096 Mar 21 17:05 ..
drwxrwxrwx  9 NAS_User NAS_User_Group 131072 Jun 28 13:33 games-setup
drwxrwxrwx 18 NAS_User NAS_User_Group 131072 Jul  3 08:19 main-backup
drwxrwxrwx  9 NAS_User NAS_User_Group 131072 Jun 28 13:33 media-hhd
drwxr-xr-x  2 root     root             4096 May 16 17:59 temp-lxc
root@pve:~# 
```

The UID and GID of this user and group is:
```sh
root@pve:~# id -u NAS_User
1000
root@pve:~# id NAS_User
uid=1000(NAS_User) gid=1000(NAS_User_Group) groups=1000(NAS_User_Group)
root@pve:~# 
root@pve:~# id NAS_User_Group
id: ‘NAS_User_Group’: no such user
root@pve:~# 
root@pve:~# getent group NAS_User_Group
NAS_User_Group:x:1000:
root@pve:~# 
```

So for the host(Proxmox OS) any user or group can read, write or execute from the mounted drives. This is important as I am restricting the access on LXC level, while my host has complete access. This also allows me to add file browser with other permissions without editing existing access from samba.

For LXC `temp-lxc`, any user or group with id *1000* is the owner who has full read write access to my drives, any other user can read the data but cannot write to it. As we configured samba with this setting I believe it is coming from there, but as my needs evolved, I changed access to suit my needs in mount drives. We can ignore it, as now all settings will be on file browser.

First, let us understand the configuration of the LXC which has Samba configured in it, in case we need to make any changes:
```sh
root@pve:~# pct config 400
arch: amd64
cores: 4
features: nesting=1
hostname: NAS-LXC
memory: 2048
mp0: /mnt/games-setup,mp=/mnt/hdd-games
mp1: /mnt/media-hhd,mp=/mnt/hdd-media
mp2: /mnt/main-backup,mp=/mnt/hdd-main-backup
net0: name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.1,hwaddr=BC:24:11:D6:4B:8B,ip=192.168.1.200/24,ip6=dhcp,type=veth
onboot: 1
ostype: ubuntu
rootfs: local-lvm:vm-400-disk-0,size=8G
swap: 1024
tags: nas
root@pve:~# 
root@pve:~# cat -n /etc/pve/lxc/400.conf
     1  arch: amd64
     2  cores: 4
     3  features: nesting=1
     4  hostname: NAS-LXC
     5  memory: 2048
     6  mp0: /mnt/games-setup,mp=/mnt/hdd-games
     7  mp1: /mnt/media-hhd,mp=/mnt/hdd-media
     8  mp2: /mnt/main-backup,mp=/mnt/hdd-main-backup
     9  net0: name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.1,hwaddr=BC:24:11:D6:4B:8B,ip=192.168.1.200/24,ip6=dhcp,type=veth
    10  onboot: 1
    11  ostype: ubuntu
    12  rootfs: local-lvm:vm-400-disk-0,size=8G
    13  swap: 1024
    14  tags: nas
root@pve:~# 
root@pve:~# 
root@pve:~# pct enter 400
root@NAS-LXC:~# 
root@NAS-LXC:~# 
root@NAS-LXC:~# 
root@NAS-LXC:~# cat -n /etc/samba/smb.conf
     1  [global]
     2     workgroup = WORKGROUP
     3     server string = %h server (Samba)
     4     netbios name = NAS-LXC
     5     security = user
     6     map to guest = Bad User
     7     guest account = nobody
     8     log file = /var/log/samba/log.%m
     9     max log size = 1000
    10     socket options = TCP_NODELAY
    11
    12  [hdd-main-backup]
    13     path = /mnt/hdd-main-backup
    14     browseable = yes
    15     read only = no
    16     guest ok = no
    17     create mask = 0777
    18     directory mask = 0777
    19     valid users = nasuser
    20     write list = nasuser
    21
    22  [hdd-media]
    23     path = /mnt/hdd-media
    24     browseable = yes
    25     read only = yes
    26     guest ok = yes
    27     create mask = 0777
    28     directory mask = 0777
    29     write list = nasuser
    30
    31  [hdd-games]
    32     path = /mnt/hdd-games
    33     browseable = yes
    34     read only = yes
    35     guest ok = yes
    36     create mask = 0777
    37     directory mask = 0777
    38     write list = nasuser
    39
root@NAS-LXC:~# 
root@NAS-LXC:~# 
root@NAS-LXC:~# 
```

The samba configurations are based on the drives and users, but it does not have any setting that can block the host or other users to write to it.
The Host is defining the mount points and has configured the LXC to read and write them, while samba has access and restrictions, nothing is moving upstream and I can add new LXC with completely different permissions.

### Using File Browser container

Let us see the requirements to use file browser container:
- having a docker based containerized system
- having the drives accessible which we want to access in file browser
- A reverse proxy handler for home-lab (caddy)
- A DNS if we want to use url instead of IP address (actual A name record)

Steps to define control:
- [[#Allow the LXC to find the drives/mount points to use]]
- [[#create the docker compose file]]
- In file browser UI
	- [[#Change admin password]]
	- [[#define guest user who you want to use for read]]
		- file browser is a bad choice with guest access
- Create a URL for the container
	- did not do, I will not be using file browser.

backup
- Take backup of container and data for safety

#### Allow the LXC to find the drives/mount points to use

As we have the `/etc/fstab` already updated to always mount the drives to fixed mount points with fixed names, we can focus on passing this mount points to the LXC(401) which has docker running in it. For this, we will update the configuration file of the LXC.
Current config file
```sh
root@pve:~# cat -n /etc/pve/lxc/401.conf
     1  arch: amd64
     2  cores: 4
     3  features: fuse=1,keyctl=1,nesting=1
     4  hostname: DockerHost
     5  memory: 2048
     6  mp0: /mnt/main-backup/homelab_backups/vikunja_lxc_backup,mp=/vikunja_backup
     7  mp1: /mnt/main-backup/homelab/apps,mp=/apps
     8  net0: name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.1,hwaddr=BC:24:11:17:FF:E8,ip=192.168.1.100/24,type=veth
     9  onboot: 1
    10  ostype: ubuntu
    11  parent: Docker_install_and_user_added
    12  rootfs: local-lvm:vm-401-disk-0,size=24G
    13  swap: 1024
    14  tags: docker
    15  unprivileged: 1
    16
    17  [Docker_install_and_user_added]
    18  #created a new user and added docker app. no docker file is added as of now
    19  arch: amd64
    20  cores: 4
    21  features: fuse=1,keyctl=1,nesting=1
    22  hostname: DockerHost
    23  memory: 2048
    24  net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=BC:24:11:17:FF:E8,ip=dhcp,ip6=dhcp,type=veth
    25  onboot: 1
    26  ostype: ubuntu
    27  rootfs: local-lvm:vm-401-disk-0,size=24G
    28  snaptime: 1778779417
    29  swap: 1024
    30  unprivileged: 1
    31  lxc.cgroup2.devices.allow: c 10:200 rwm
    32  lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
root@pve:~# 
```

Add the drives as mount points to this config file in next available mount points (mp2, mp3, mp4):
```sh
root@pve:~# cat -n /etc/pve/lxc/401.conf
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
root@pve:~# 
```

Check that LXC should not have the files at this moment:
```sh
root@DockerHost:~# ls -al /mnt/
total 8
drwxr-xr-x  2 root root 4096 May  7  2024 .
drwxr-xr-x 24 root root 4096 Jun 28 08:04 ..
root@DockerHost:~# 
```

Now that the config is updated, we need to **restart the LXC** for the new files to be mounted.
```sh
root@pve:~# pct list
VMID       Status     Lock         Name                
400        running                 NAS-LXC             
401        running                 DockerHost          
root@pve:~# pct reboot 401
root@pve:~# pct list
VMID       Status     Lock         Name                
400        running                 NAS-LXC             
401        running                 DockerHost          
root@pve:~# pct enter 401
root@DockerHost:~# ls -al /mnt/
total 392
drwxr-xr-x  5 root   root      4096 Jul  4 14:25 .
drwxr-xr-x 24 root   root      4096 Jul  4 14:25 ..
drwxrwxrwx  9 nobody nogroup 131072 Jun 28 08:03 hdd-games
drwxrwxrwx 18 nobody nogroup 131072 Jul  3 02:49 hdd-main-backup
drwxrwxrwx  9 nobody nogroup 131072 Jun 28 08:03 hdd-media
root@DockerHost:~# 
root@DockerHost:~# id nobody  
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
root@DockerHost:~# getent group nogroup
nogroup:x:65534:
root@DockerHost:~# 
```

Now, the drives are available for editing in the LXC, we can see that the user is `nobody` with `nogroup` as the user. As all users have access to read, write and execute, we can directly use this for our docker. If I will add restrictions, I will have to plan how to update other mp0 and mp1 changes, as they are used by [[All Published Notes/Homelab/010 Self hosted Task Manager - Vikunja\|vikunja]].

#### create the docker compose file

The docker file will use the image uploaded on the [docker page](https://hub.docker.com/r/filebrowser/filebrowser) or you can find it in [github](https://github.com/filebrowser/filebrowser#project-status).
The file will need a database and a configuration file, we can create a directory which will have all data related to our container:
```sh
root@DockerHost:~# ls -al /apps
total 260
drwxrwxrwx  3 nobody nogroup 131072 Jun 13 11:56 .
drwxr-xr-x 24 root   root      4096 Jul  4 14:25 ..
drwxrwxrwx  4 nobody nogroup 131072 Jun 19 19:01 localDns
root@DockerHost:~# mkdir -p /apps/file_browser
root@DockerHost:~# sudo chown -R 65534:65534 /apps/file_browser
chown: changing ownership of '/apps/file_browser': Operation not permitted
root@DockerHost:~# ls -al /apps
total 388
drwxrwxrwx  4 nobody nogroup 131072 Jul  4 14:57 .
drwxr-xr-x 24 root   root      4096 Jul  4 14:25 ..
drwxrwxrwx  2 nobody nogroup 131072 Jul  4 14:57 file_browser
drwxrwxrwx  4 nobody nogroup 131072 Jun 19 19:01 localDns
root@DockerHost:~# 
```

As `/apps` is a folder inside `/mnt/main-backup/homelab/apps`, we need not do `chown` operation inside it. Now, we can use this folder to define and store all files for our file browser container.
Create the files we want our container to use:
```sh
root@DockerHost:~# ls -al /apps/file_browser/
total 256
drwxrwxrwx 2 nobody nogroup 131072 Jul  4 14:57 .
drwxrwxrwx 4 nobody nogroup 131072 Jul  4 14:57 ..
root@DockerHost:~# touch /apps/file_browser/filebrowser.db
root@DockerHost:~# touch /apps/file_browser/settings.json 
root@DockerHost:~# ls -al /apps/file_browser/
total 256
drwxrwxrwx 2 nobody nogroup 131072 Jul  4 15:34 .
drwxrwxrwx 4 nobody nogroup 131072 Jul  4 14:57 ..
-rwxrwxrwx 1 nobody nogroup      0 Jul  4 15:33 filebrowser.db
-rwxrwxrwx 1 nobody nogroup      0 Jul  4 15:34 settings.json
root@DockerHost:~# 
```

The yml file:
```yml
services:
  filebrowser: # name of the service
    image: filebrowser/filebrowser:latest # name of the user and file in docker hub
    container_name: filebrowser # name for the container in our machine
    user: "65534:65534" # execute as nobody:nogroup
    ports:
      - "10002:80" # use port 10002 for communication
    security_opt:
      - no-new-privileges:true # cannot change permission from inside the container
    read_only: true # container system root is read-only
    tmpfs:
      - /tmp:size=10M,uid=65534,gid=65534 # A small memory for the container to write its temp files into
      - /config:size=1M,uid=65534,gid=65534 # Gives the boot script a writeable sandbox in RAM for setup
    environment:
      - FB_DATABASE=/database/filebrowser.db # The database file to be used by filebrowser
      - FB_CONFIG=/database/settings.json # The configuration file to be used by filebrowser
    volumes:
      # Hard drives (Fully Read/Write)
      - /mnt/hdd-games:/srv/storage/games-setup
      - /mnt/hdd-media:/srv/storage/media-hhd
      - /mnt/hdd-main-backup:/srv/storage/main-backup
      # Persistent App Data Folder
      - /apps/file_browser:/database
    restart: unless-stopped
```

Here, `/tmp:size=10M,uid=65534,gid=65534` and `/config:size=1M,uid=65534,gid=65534` are with uid and gid, which allows the desired non-root user to read and write the files.

If done correctly, the 'admin' username and password is created as a random password and the logs can be used to see the password:
```sh
root@DockerHost:~# docker logs filebrowser | grep -i "password"
cp: can't preserve ownership of '/config/settings.json': Operation not permitted
2026/07/04 15:47:30 Using config file: /config/settings.json
2026/07/04 15:47:30 WARNING: filebrowser.db can't be found. Initialing in /database/
2026/07/04 15:47:30 Using database: /database/filebrowser.db
2026/07/04 15:47:30 Performing quick setup
2026/07/04 15:47:30 User 'admin' initialized with randomly generated password: A8ks245644e_-bY6
root@DockerHost:~# 
```

So the default password generated is `A8ks245644e_-bY6`.
We can change it in UI.

Open the LXC ip and use port '10002' in a browser, for e.g., my IP is '192.168.1.100/' so I will use : `http://192.168.1.100:10002/`.

#### Change admin password
In the browser, open the dashboard `http://192.168.1.100:10002/`, open the profile page:
![054 homelab file browser profile page.png](/img/user/All%20Published%20Notes/Homelab/Images/054%20homelab%20file%20browser%20profile%20page.png)
This allows us to change current user password.

We can also open user management as admin:
![055 homelab file browser user page.png](/img/user/All%20Published%20Notes/Homelab/Images/055%20homelab%20file%20browser%20user%20page.png)
This allows us to edit the data as an admin:
![056 homelab file browser admin user management.png](/img/user/All%20Published%20Notes/Homelab/Images/056%20homelab%20file%20browser%20admin%20user%20management.png)

Users can be added or deleted when we visit user management page as an admin, I will add new user who has all read write access as well as allow guest for read only access:
Create a new user:
![057 homelab file browser new user.png](/img/user/All%20Published%20Notes/Homelab/Images/057%20homelab%20file%20browser%20new%20user.png)

Define the user data:
![058 homelab file browser user settings.png](/img/user/All%20Published%20Notes/Homelab/Images/058%20homelab%20file%20browser%20user%20settings.png)

As this is the main user with password, we want to give it complete access to storage directory:
1. user name
2. password (in setting, minimum 12 characters)
3. scope: the directories user has access to
	1. use path like `/storage` instead of `/srv/storage/` for all directories
	2. multiple paths are not supported
4. can the user change their own password
5. permissions

This works when we want to access the files as a registered user.

We cannot use comma separated folders, one user can only be allowed to access a single scope:
![059 homelab file browser wrong scope.png](/img/user/All%20Published%20Notes/Homelab/Images/059%20homelab%20file%20browser%20wrong%20scope.png)

When we try to save with multiple scope, the saving option will fail.

#### define guest user who you want to use for read

Ideally, file browser does not allow us to define a scope with multiple directories, it also does not allow to access data without sign in.
Now, we have 2 options for accessing data as guest:
- we can define 2 users who has access to `storage/games-setup` and `storage/media-hhd` respectively. 
- we can change our volumes in docker's yml file to create a common directory which has both drives(`storage/games-setup` and `storage/media-hhd`) and another which has the `storage/main-backup` drive.
	- see

##### create 2 guest users
To avoid re-writing the whole yml, I will create 2 new users, but having a long password and sharing it with all guests will be a pain, so we will create passwords which are one character long.

Update the global settings to allow new users to be created with one character long passwords:
![060 homelab filelab password length.png](/img/user/All%20Published%20Notes/Homelab/Images/060%20homelab%20filelab%20password%20length.png)

Create the guest users with media as scope:
using 'a' as a password
![061 homelab filelab guest with small password.png](/img/user/All%20Published%20Notes/Homelab/Images/061%20homelab%20filelab%20guest%20with%20small%20password.png)

using 'guest' as a password:
![062 homelab filelab guest with simple password.png](/img/user/All%20Published%20Notes/Homelab/Images/062%20homelab%20filelab%20guest%20with%20simple%20password.png)

As trying to create a guest account with a simple password fails, I will not be using it anymore
##### update yml file
update the yml file like this:
```yml
volumes:
      # Guest-Accessible Hard Drives (Grouped under a single 'public' directory)
      - /mnt/hdd-games:/srv/storage/public/games-setup
      - /mnt/hdd-media:/srv/storage/public/media-hhd
      
      # Isolated Private Hard Drive (Kept outside 'public')
      - /mnt/hdd-main-backup:/srv/storage/main-backup
      
      # Persistent App Data Folder
      - /apps/file_browser:/database
```

now, you can create a guest with scope as `storage/public` and only read permission

> [!Warning]
> I could not create a simple guest access, this was an important need for me to allow access to others without giving a login or write access.
> I will find another, but if you are comfortable to create user credentials, this is a very good choice.


### Alternative
- kodbox: A good alternative, but most of the documents seem to be in chinese, so edge case handling can be a problem.
	- https://github.com/kalcaddle/kodbox
	- https://hub.docker.com/r/kodcloud/kodbox
- FileStash
	- [[All Published Notes/Homelab/017 Using Filestash to access Samba share\|017 Using Filestash to access Samba share]]





---

[^1]:
[^2]:

