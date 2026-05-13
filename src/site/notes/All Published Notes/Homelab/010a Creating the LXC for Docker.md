---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/010a-creating-the-lxc-for-docker/"}
---

created: 2026-05-13
updated: 2026-05-13

For Docker, we want more Cores and RAM for ensuring smooth processing, so we use 4 cores and 2 GB RAM. 
We also want to keep the container unprivileged, this ensures security by keeping the LXC limited in access to the machine. 

![032 Ubuntu container.png](/img/user/All%20Published%20Notes/Homelab/Images/032%20Ubuntu%20container.png)

this leads to the container created with the settings as added:
```sh
()

Logical volume "vm-401-disk-0" created.  
Logical volume pve/vm-401-disk-0 changed.  
Creating filesystem with 6291456 4k blocks and 1572864 inodes  
Filesystem UUID: 87a1a2be-dd44-4029-8439-d0181e220619  
Superblock backups stored on blocks:  
32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,  
extracting archive '/var/lib/vz/template/cache/ubuntu-24.04-standard_24.04-2_amd64.tar.zst'  
Total bytes read: 564490240 (539MiB, 452MiB/s)  
Detected container architecture: amd64  
Setting up 'proxmox-regenerate-snakeoil.service' to regenerate snakeoil certificate..  
Creating SSH host key 'ssh_host_ed25519_key' - this may take some time ...  
done: SHA256:gVOdbEVZ39HBjwCAdx9dC9Xd3F+ZmXnEoTaTT+X8QsY root@DockerHost  
Creating SSH host key 'ssh_host_ecdsa_key' - this may take some time ...  
done: SHA256:Nww89F+GSYciRUDMSBIuQox/iMbj77crBQmYJZkdlaQ root@DockerHost  
Creating SSH host key 'ssh_host_rsa_key' - this may take some time ...  
done: SHA256:MO4wvFqzdGXQrlCQgt53eMwYktHGnzBLVm29YJHuuqA root@DockerHost  
TASK OK
```

- **Nesting:** Must be **Enabled**. (Allows Docker to create its own child namespaces).
- **keyctl:** Must be **Enabled**. (Prevents issues with Docker’s credential management).
- **FUSE:** Optional, but helpful if you plan on using certain storage drivers or mounting cloud drives.

To enable these options, once the LXC is created and not yet started, open Options => Features and double click it to get the other options.

![033 LXC Features.png](/img/user/All%20Published%20Notes/Homelab/Images/033%20LXC%20Features.png)

Now we have an LXC ready for using with Docker.

> [!Note]
> I had setup my Proxmox node to use the DNS as the host's IP address fetched from Netbird, this stops new LXC from connecting to internet as they are not on same NetBird account. Had to go there and change DNS to fallback on 1.1.1.1 and 8.8.8.8 when needed.


First as all tutorials say, "Let there be light" or `root@DockerHost:~# apt update && apt upgrade -y` in the terminal of this LXC:
```sh
root@pve:~# pct enter 401
root@DockerHost:~# apt update && apt upgrade -y
```

After the update, we can try to install apps like 
- curl
- netbird
- Fresh
- etc.
This is to ensure the LXC has the same work flow when we need to SSH into this and when we are debugging, as we do not want new errors at that time.



---

[^1]:
[^2]:

