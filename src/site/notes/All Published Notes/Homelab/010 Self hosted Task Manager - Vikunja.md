---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/010-self-hosted-task-manager-vikunja/"}
---

created: 2026-05-06
updated: 2026-05-15

There are lot of alternatives to TODO and task tracking. 
I read through the available options and decided to go with "[Vikunja](https://vikunja.io/)". This is overkill, but will be useful as my needs expand in future. It will also be a good entry point for Docker.

I do not currently know anything about docker other than the memes, so I will make a few small projects for testing and learning[^1] before installing Vikunja.

Docker Learning:
- [[All Published Notes/Homelab/010a Creating the LXC for Docker\|010a Creating the LXC for Docker]]
- [[All Published Notes/Homelab/010b Installing Docker on LXC\|010b Installing Docker on LXC]]
	- [[All Published Notes/Homelab/010b Installing Docker on LXC#Docker User (Highly Suggested for automation)\|Docker User]]
- [[All Published Notes/Homelab/010c opening Docker in host machine\|010c opening Docker in host machine]]
- 010d installing DB and accessing it from docker -> postponed for now.

## Installing Vikunja with postgres SQL

The files need a folder in which they will store all data to avoid loosing all info when containers are re-booted:
```sh
root@DockerHost:~# mkdir -p /opt/vikunja/files # for Vikunja files
root@DockerHost:~# mkdir -p /opt/vikunja/db # for postgres SQL
```

#### Vikunja
When we download the 'Vikunja' image and create a container, a user with *UID 1000* is considered as the main user, so we want the user to have access to these directories for read and write as this user does not have root access at any point.
As we have created a user 'dockeruser'(*UID 2300*) in a group called docker(*GID 990*), we will give this UID the ownership of the directories we want to use:
```sh
# Give Vikunja permissions (it runs as user 1000 by default)
root@DockerHost:~# chown -R 2300:990 /opt/vikunja/files
```

My current users:
```sh
root@DockerHost:~# printf "%-20s %-8s %-8s %-20s %-20s\n" "USER" "UID" "GID" "PRIMARY GROUP" "SHELL"
printf "%-20s %-8s %-8s %-20s %-20s\n" "----" "---" "---" "-------------" "-----"
for user in $(getent passwd | cut -d: -f1); do
    uid=$(getent passwd $user | cut -d: -f3)
    gid=$(getent passwd $user | cut -d: -f4)
    group=$(getent group $gid | cut -d: -f1)
    shell=$(getent passwd $user | cut -d: -f7)
    printf "%-20s %-8s %-8s %-20s %-20s\n" "$user" "$uid" "$gid" "$group" "$shell"
done
USER                 UID      GID      PRIMARY GROUP        SHELL               
----                 ---      ---      -------------        -----               
root                 0        0        root                 /bin/bash           
daemon               1        1        daemon               /usr/sbin/nologin   
bin                  2        2        bin                  /usr/sbin/nologin   
sys                  3        3        sys                  /usr/sbin/nologin   
sync                 4        65534    nogroup              /bin/sync           
games                5        60       games                /usr/sbin/nologin   
man                  6        12       man                  /usr/sbin/nologin   
lp                   7        7        lp                   /usr/sbin/nologin   
mail                 8        8        mail                 /usr/sbin/nologin   
news                 9        9        news                 /usr/sbin/nologin   
uucp                 10       10       uucp                 /usr/sbin/nologin   
proxy                13       13       proxy                /usr/sbin/nologin   
www-data             33       33       www-data             /usr/sbin/nologin   
backup               34       34       backup               /usr/sbin/nologin   
list                 38       38       list                 /usr/sbin/nologin   
irc                  39       39       irc                  /usr/sbin/nologin   
_apt                 42       65534    nogroup              /usr/sbin/nologin   
nobody               65534    65534    nogroup              /usr/sbin/nologin   
systemd-network      998      998      systemd-network      /usr/sbin/nologin   
systemd-timesync     997      997      systemd-timesync     /usr/sbin/nologin   
dhcpcd               100      65534    nogroup              /bin/false          
messagebus           101      101      messagebus           /usr/sbin/nologin   
syslog               102      102      syslog               /usr/sbin/nologin   
systemd-resolve      992      992      systemd-resolve      /usr/sbin/nologin   
sshd                 103      65534    nogroup              /usr/sbin/nologin   
postfix              104      105      postfix              /usr/sbin/nologin   
uuidd                105      107      uuidd                /usr/sbin/nologin   
tcpdump              106      109      tcpdump              /usr/sbin/nologin   
dockeruser           2300     990      docker               /bin/bash           
root@DockerHost:~# 

```
#### postgres
When we download the 'postgres' image and create a container, the official Postgres image usually starts as **root**, performs internal setup, and then _automatically_ changes the ownership of the `/var/lib/postgresql/data` folder to its own internal `postgres` user(UID 999 inside the docker container). This means, we can skip giving access to the directory it will use as the docker user made itself part of the group which has ownership. But to avoid issues and read write permissions, we can give access as a safety measure.

**Postgres also does not work very well with different user group**, so we initially tried to set the group and user using the LXC but had some issues. So we want to set the user permissions using the host proxmox machine and a offset of 100k (this is because of some setup logic between uid and gid in host and LXCs).

```sh
root@pve:~# pct enter 401
root@DockerHost:~# ls -al /opt/vikunja/files
total 8
drwxr-xr-x 2 root root 4096 May 15 20:39 .
drwxr-xr-x 4 root root 4096 May 15 20:39 ..
root@DockerHost:~# ls -al /opt/vikunja/db   
total 136
drwx------ 19  999 systemd-journal  4096 May 16 12:16 .
drwxr-xr-x  4 root root             4096 May 15 20:39 ..
-rw-------  1  999 systemd-journal     3 May 15 20:49 PG_VERSION
drwx------  6  999 systemd-journal  4096 May 15 20:49 base
drwx------  2  999 systemd-journal  4096 May 16 12:17 global
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_commit_ts
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_dynshmem
-rw-------  1  999 systemd-journal  5743 May 15 20:49 pg_hba.conf
-rw-------  1  999 systemd-journal  2640 May 15 20:49 pg_ident.conf
drwx------  4  999 systemd-journal  4096 May 16 12:21 pg_logical
drwx------  4  999 systemd-journal  4096 May 15 20:49 pg_multixact
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_notify
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_replslot
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_serial
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_snapshots
drwx------  2  999 systemd-journal  4096 May 16 12:16 pg_stat
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_stat_tmp
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_subtrans
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_tblspc
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_twophase
drwx------  3  999 systemd-journal  4096 May 15 20:49 pg_wal
drwx------  2  999 systemd-journal  4096 May 15 20:49 pg_xact
-rw-------  1  999 systemd-journal    88 May 15 20:49 postgresql.auto.conf
-rw-------  1  999 systemd-journal 29829 May 15 20:49 postgresql.conf
-rw-------  1  999 systemd-journal    24 May 16 12:16 postmaster.opts
-rw-------  1  999 systemd-journal    94 May 16 12:16 postmaster.pid
root@DockerHost:~# exit
exit
root@pve:~# # This makes the LXC's entire filesystem visible at /mnt/temp-lxc
root@pve:~# mkdir -p /mnt/temp-lxc
root@pve:~# pct mount 401 /mnt/temp-lxc
400 too many arguments
pct mount <vmid>
root@pve:~# pct mount 401 
mounted CT 401 in '/var/lib/lxc/401/rootfs'
root@pve:~# ls -al /var/lib/lxc/401/rootfs
total 96
drwxr-xr-x 21 100000 100000  4096 May 16 17:46 .
drwxr-xr-x  5 root   root    4096 May 16 17:46 ..
lrwxrwxrwx  1 100000 100000     7 Apr 22  2024 bin -> usr/bin
drwxr-xr-x  2 100000 100000  4096 Apr  8  2024 bin.usr-is-merged
drwxr-xr-x  2 100000 100000  4096 Apr 22  2024 boot
drwxr-xr-x  2 100000 100000  4096 Apr 22  2024 dev
drwxr-xr-x 84 100000 100000  4096 May 16 17:46 etc
drwxr-xr-x  2 100000 100000  4096 May 14 22:29 home
lrwxrwxrwx  1 100000 100000     7 Apr 22  2024 lib -> usr/lib
lrwxrwxrwx  1 100000 100000     9 May  7  2024 lib32 -> usr/lib32
lrwxrwxrwx  1 100000 100000     9 Apr 22  2024 lib64 -> usr/lib64
drwxr-xr-x  2 100000 100000  4096 Apr  8  2024 lib.usr-is-merged
lrwxrwxrwx  1 100000 100000    10 May  7  2024 libx32 -> usr/libx32
drwx------  2 root   root   16384 May 13 23:04 lost+found
drwxr-xr-x  2 100000 100000  4096 May  7  2024 media
drwxr-xr-x  2 100000 100000  4096 May  7  2024 mnt
drwxr-xr-x  4 100000 100000  4096 May 16 02:09 opt
drwxr-xr-x  2 100000 100000  4096 Apr 22  2024 proc
drwx------  6 100000 100000  4096 May 16 01:06 root
drwxr-xr-x 10 100000 100000  4096 May  7  2024 run
lrwxrwxrwx  1 100000 100000     8 Apr 22  2024 sbin -> usr/sbin
drwxr-xr-x  2 100000 100000  4096 Mar 31  2024 sbin.usr-is-merged
drwxr-xr-x  2 100000 100000  4096 May  7  2024 srv
drwxr-xr-x  2 100000 100000  4096 Apr 22  2024 sys
drwxrwxrwt  8 100000 100000  4096 May 16 17:46 tmp
drwxr-xr-x 14 100000 100000  4096 May  7  2024 usr
drwxr-xr-x 11 100000 100000  4096 May 13 23:16 var
root@pve:~# ls -al /var/lib/lxc/401/rootfs/opt/vikunja/
total 16
drwxr-xr-x  4 100000 100000 4096 May 16 02:09 .
drwxr-xr-x  4 100000 100000 4096 May 16 02:09 ..
drwx------ 19 100999 100999 4096 May 16 17:46 db
drwxr-xr-x  2 100000 100000 4096 May 16 02:09 files
root@pve:~# chown -R 102300:100990 /var/lib/lxc/401/rootfs/opt/vikunja/files/
root@pve:~# chown -R 102300:100990 /var/lib/lxc/401/rootfs/opt/vikunja/files
root@pve:~# chmod -R 775 /var/lib/lxc/401/rootfs/opt/vikunja/files
root@pve:~# # Fix Postgres files (mapping internal 999:999 -> host 100999:100999)
root@pve:~# chown -R 100999:100999 /var/lib/lxc/401/rootfs/opt/vikunja/db
root@pve:~# chmod -R 700 /var/lib/lxc/401/rootfs/opt/vikunja/db
root@pve:~# pct unmount 401
root@pve:~# 
root@pve:~# pct enter 401
root@DockerHost:~# hostname -I
192.168.1.100 172.17.0.1 172.18.0.1 100.77.8.142 
root@DockerHost:~# 
```

Now, we mounted the LXC's folders into the proxmox host, changed the owners(`chown`) with an offset of '100000' and now our docker user can easily access the folders as well as postgres will not encounter issues for running.


> [!Note]
> We can also use tool like **pgAdmin** or **CloudBeaver** if we want to see the DB info in a UI application, but this is not in current scope

#### portainer stack

Now, we can create a compose file or stack (portainer terminology) :

```yml
services:
  db:
    image: postgres:16-alpine
    restart: always
    # This forces Postgres to run safely inside the container while matching standard Docker patterns 
    user: "999:999"
    environment:
      POSTGRES_USER: vikunja
      POSTGRES_PASSWORD: your_strong_password  # Change this!
      POSTGRES_DB: vikunja
    volumes:
      - /opt/vikunja/db:/var/lib/postgresql/data # Mapping docker internal folder/file to our LXC folder/file

  vikunja:
    image: vikunja/vikunja
    user: "2300:990" # LXC user and group
    restart: always
    environment:
      VIKUNJA_DATABASE_HOST: db
      VIKUNJA_DATABASE_TYPE: postgres
      VIKUNJA_DATABASE_USER: vikunja
      VIKUNJA_DATABASE_PASSWORD: your_strong_password # Must match above!
      VIKUNJA_DATABASE_DATABASE: vikunja
      VIKUNJA_SERVICE_PUBLICURL: http://YOUR_LXC_IP:3456/ # Change?
      VIKUNJA_SERVICE_ENABLEREGISTRATION: "true" # Change to 'false' after user is registered
    volumes:
      - /opt/vikunja/files:/app/vikunja/files
    ports:
      - "3456:3456"
    depends_on:
      - db
```

Now, we need to add this stack to portainer so that we can make the containers which do not need port mapping to communicate (dedicated 'docker network' using single 'stack' in portainer)

**Create Portainer Stack**
- Log into Portainer Dashboard.
- if you are in home page, click on the environment in which you want to create the container, e.g., 'local'
- Go to **Stacks** > *Add stack*.
- Name it `vikunja`.
- Paste the above configuration file.

portainer home page:
![036 portainer home page.png](/img/user/All%20Published%20Notes/Homelab/Images/036%20portainer%20home%20page.png)

**Deploy and Access**
1. Click **Deploy the stack**.
2. Wait about 30 seconds for Postgres to initialize.
3. Open your browser and go to `http://<YOUR_LXC_IP>:3456`.

Assuming my LXC has an IP address of `192.168.1.100`, we can write the file as:

```yml
services:
  db:
    image: postgres:16-alpine
    restart: always
    user: "999:999"
    environment:
      POSTGRES_USER: vikunja
      POSTGRES_PASSWORD: "P@$w0rD!"
      POSTGRES_DB: vikunja
    volumes:
      - /opt/vikunja/db:/var/lib/postgresql/data 

  vikunja:
    image: vikunja/vikunja
    user: "2300:990"
    restart: always
    environment:
      VIKUNJA_DATABASE_HOST: db
      VIKUNJA_DATABASE_TYPE: postgres
      VIKUNJA_DATABASE_USER: vikunja
      VIKUNJA_DATABASE_PASSWORD: "P@$w0rD!"
      VIKUNJA_DATABASE_DATABASE: vikunja
      VIKUNJA_SERVICE_PUBLICURL: http://192.168.1.100:3456/ 
      VIKUNJA_SERVICE_ENABLEREGISTRATION: "true"
    volumes:
      - /opt/vikunja/files:/app/vikunja/files
    ports:
      - "3456:3456"
    depends_on:
      - db
```

after deploying, I need to access it.

#### Accessing
We need to access the same IP address we have for the LXC and change port to 3456 for accessing vikunja: `http://192.168.1.100:3456`.
Note the use of 'http' instead of 'https', as we do not have any SSL certificates.

##### Admin
Once the dashboard is accessible, create an account, this is your admin account, so save it properly.
if you want to enable more users to login, you can log them in before blocking registrations, if registration is needed, you will have to un-block and allow that.

### Blocking registrations on vikunja
As we do not want anyone to login and use vikunja, we can block registrations by updating the environment variables of vikunja image in the stack, specifically `VIKUNJA_SERVICE_ENABLEREGISTRATION: "false"`.
Update the stack and restart, you will retain the data (as we used volumes to store data outside the docker) but change possibility of registration.

### Using vikunja
you can add tasks as current tasks from home page or create a project and track tasks there.







---

[^1]: https://www.kdnuggets.com/5-fun-docker-projects-for-absolute-beginners
[^2]:

