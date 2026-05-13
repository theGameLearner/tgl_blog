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

### Suggestion
Create a user who manages docker ` usermod -aG docker $USER` rather than using 'root' user to handle docker.





---

[^1]: [[All Published Notes/Homelab/010a Creating the LXC for Docker\|010a Creating the LXC for Docker]]
[^2]:

