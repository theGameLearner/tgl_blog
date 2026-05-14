---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/010b-installing-docker-on-lxc/"}
---

created: 2026-05-13
updated: 2026-05-14

Installing Docker on an LXC will happen in steps:

## System Preparation
To install Docker on an LXC (401 created before[^1]), we first need to verify the certificates(ca-certificates) of the sites we are using, ability to fetch data using URL(curl), ability to verify the app signature(gnupg) as well as a way for scripts to know our LXC Base distribution information (lsb_release). We can achieve all this packages by the following:
```sh
apt install -y ca-certificates curl gnupg lsb-release
```

## Add Docker's GPG Key
A GPG key lets us download updates as they arrive, so we want to be able to get new updates when needed.

```sh
$ mkdir -p /etc/apt/keyrings # folder to store the keys
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg # add the key to the written path
```

Now that we have the keyring file, we need to add the official Docker repository to our Ubuntu system's APT sources list:
```sh
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

This creates a new file (`/etc/apt/sources.list.d/docker.list`) that tells `apt` where to find Docker packages and how to verify they're authentic.
- The line `[arch=$(...docker.gpg]` ensures that we download a version that is compatible with our CPU's version rather than a software that we cannot run. 
- The line `signed-by=...` ensures we get a signed file which matches our signed keyring.
- The line `https://download.docker.com/linux/ubuntu` ensures we are downloading for ubuntu
The rest can be understood as ensuring we get an address of stable version of docker which is compatible with our machine.

Now we are ready to install Docker

## Install Docker
Docker is not a single package but a few packages that are together make the docker stack:

```sh
apt update && apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

After this, your machine has Docker.

## Post installation
**Storage Driver Check**
Confirm that we are using 'overlay2' storage: 
```sh
root@DockerHost:~# docker info | grep Storage
 Storage Driver: overlayfs
root@DockerHost:~#
```
If we are using 'vfs' that means that we are not having the 'nesting' feature enabled in the LXC. Underlying file system Ext4 and ZFS usually do support overlay but ZFS may need some other changes which I cannot check.

Now your docker is ready to be used. 

### Optional Post installation:
#### Docker User (Highly Suggested for automation)
We want docker to be managed by a user, but root should not manage directly as it can influence what the LXC can do. (`adduser dockeruser`)

We get a pre-created user group called `docker` due to `docker-ce` and can add user to it for accessing docker. (`usermod -aG docker dockeruser`)

We also want our user to be able to run the processes whether we are logged in via SSH or automating any action through this user, so we give it permission to linger. (`loginctl enable-linger dockeruser`)

Why do we want our user to linger?
- **The Problem**: Your Docker containers, even if run in detached mode (`-d`), are child processes of your user's session. When `systemd` cleans up, it sends a `SIGTERM` signal, causing your containers and the Rootless Docker daemon to shut down gracefully.
- **The Solution (`enable-linger`)**: This command tells `systemd`: "For the user `dockeruser`, do **not** perform this cleanup. Keep its user-specific service manager running, even when the user has no active login sessions".
- **The Result**: The `systemd --user` instance for `dockeruser` starts at boot and stays running. This allows your Rootless Docker daemon (managed as a `systemd --user` service) to start automatically and keep your containers alive continuously.

reference logs:
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
root@DockerHost:~# 
root@DockerHost:~# 
root@DockerHost:~# 
root@DockerHost:~# # created a  user 'dockeruser'(2300) and added it to a group called 'dockeradmins'(2300) for future use
root@DockerHost:~# 
root@DockerHost:~# id dockeruser
uid=2300(dockeruser) gid=2300(dockeradmins) groups=2300(dockeradmins)
root@DockerHost:~# 
root@DockerHost:~# 
root@DockerHost:~# printf "%-20s %-8s %-8s %-20s %-20s\n" "USERNAME" "UID" "GID" "GROUP" "SHELL"
printf "%-20s %-8s %-8s %-20s %-20s\n" "--------" "---" "---" "-----" "-----"
for user in $(getent passwd | cut -d: -f1); do
    user_info=$(getent passwd $user)
    username=$(echo $user_info | cut -d: -f1)
    uid=$(echo $user_info | cut -d: -f3)
    gid=$(echo $user_info | cut -d: -f4)
    shell=$(echo $user_info | cut -d: -f7)
    groupname=$(getent group $gid | cut -d: -f1)
    printf "%-20s %-8s %-8s %-20s %-20s\n" "$username" "$uid" "$gid" "$groupname" "$shell"
done
USERNAME             UID      GID      GROUP                SHELL               
--------             ---      ---      -----                -----               
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
dockeruser           2300     2300     dockeradmins         /bin/bash           
root@DockerHost:~# 
root@DockerHost:~# 
root@DockerHost:~# # enable linger on the new user creaeted, this should not give a response but should work
root@DockerHost:~# loginctl enable-linger dockeruser
root@DockerHost:~# 
```

> [!Warning]
> The package `docker-ce` automatically adds a group called 'docker'(GID: 990) for using and accessing docker, it is simpler and error-free to use this group rather than a custom group like 'dockeradmins'.


##### Docker User permissions

Check the owner and permissions of '/var/run/docker.sock' file which is essential for docker:
```sh
root@DockerHost:~# # see the list of installed packages that are docker related
root@DockerHost:~# apt list --installed | grep docker

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

docker-buildx-plugin/noble,now 0.34.0-1~ubuntu.24.04~noble amd64 [installed,automatic]
docker-ce-cli/noble,now 5:29.4.3-1~ubuntu.24.04~noble amd64 [installed]
docker-ce-rootless-extras/noble,now 5:29.4.3-1~ubuntu.24.04~noble amd64 [installed,automatic]
docker-ce/noble,now 5:29.4.3-1~ubuntu.24.04~noble amd64 [installed]
docker-compose-plugin/noble,now 5.1.3-1~ubuntu.24.04~noble amd64 [installed]
root@DockerHost:~# 
root@DockerHost:~# # check the file exists
root@DockerHost:~# ls /var/run/docker.sock
/var/run/docker.sock
root@DockerHost:~# 
root@DockerHost:~# # check the file permissions
root@DockerHost:~# ls -al /var/run/docker.sock
srw-rw---- 1 root docker 0 May 14 16:07 /var/run/docker.sock
root@DockerHost:~# 
root@DockerHost:~# # check the group exists
root@DockerHost:~# getent group docker
docker:x:990:
root@DockerHost:~# 
root@DockerHost:~# # add the user to the docker group
root@DockerHost:~# usermod -aG docker dockeruser
root@DockerHost:~# getent group docker
docker:x:990:dockeruser
root@DockerHost:~# 
root@DockerHost:~# 
```

Now, the user should have the essential permissions for running and controlling docker:
```sh
root@DockerHost:~# # switch to the created user
root@DockerHost:~# su - dockeruser
su: warning: cannot change directory to /home/dockeruser: No such file or directory
dockeruser@DockerHost:/root$ 
dockeruser@DockerHost:/root$ 
dockeruser@DockerHost:/root$ docker version
Client: Docker Engine - Community
 Version:           29.4.3
 API version:       1.54
 Go version:        go1.26.2
 Git commit:        055a478
 Built:             Wed May  6 17:07:36 2026
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          29.4.3
  API version:      1.54 (minimum version 1.40)
  Go version:       go1.26.2
  Git commit:       56be731
  Built:            Wed May  6 17:07:36 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v2.2.3
  GitCommit:        77c84241c7cbdd9b4eca2591793e3d4f4317c590
 runc:
  Version:          1.3.5
  GitCommit:        v1.3.5-0-g488fc13e
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
dockeruser@DockerHost:/root$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
dockeruser@DockerHost:/root$ exit
logout
root@DockerHost:~# 
root@DockerHost:~# 
```

As seen, we can use 'dockeruser' to run docker commands now.


### Other Suggestions
- **Docker Compose:** Always use Docker Compose. It allows you to define your infrastructure as code. Create a directory for each service (e.g., `~/docker/nginx/`) and keep your `docker-compose.yml` there.
- **Port Management:** Remember that the LXC has its own IP address. If you run an Nginx container on port 80 inside the LXC, it will be reachable at `http://[LXC-IP]:80`.
- **Backups:** Since you are in Proxmox, use the **Backup** feature. Before making major changes to your Docker setup, take a "Snapshot" of the LXC. If Docker breaks the networking or storage, you can roll back in seconds.





---

[^1]: [[All Published Notes/Homelab/010a Creating the LXC for Docker\|010a Creating the LXC for Docker]]
[^2]: [[All Published Notes/Homelab/010b Installing Docker on LXC\|010b Installing Docker on LXC]]


