---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/010c-opening-docker-in-host-machine/"}
---

created: 2026-05-14
updated: 2026-05-15

We have got Docker running on our LXC (ubuntu LXC ID:401 and can be managed by user 'dockeruser' in group 'dockeradmins') inside the Proxmox OS.
We want this Docker application to be accessible from the main machine (linux mint) to avoid having to SSH into the LXC for running all applications and to debug. It also helps that there are many UI applications to control and interact with docker for simple steps.
How to connect?
- **Docker CLI Contexts**: This allows me to connect in terminal to access and control the machine with CLI commands. 
	- I will not be exploring it as I have saved connections in my 'Tabby' terminal that makes CLI into docker very easy.
- **Portainer**: install this on the LXC and you can connect to this through a browser using the IP address, as I am on a netbird mesh, this is a direct access without so many authentication limitations
- **Dockge**: Similar to portainer but more simple for people that know what they are doing.
	- As an absolute beginner, I will opt out of this for now.

## Setting up portainer on LXC

Portainer needs access to a volume(storage) for storing the data, so we create one named 'portainer_data': `docker volume create portainer_data`.
We want to run portainer with docker, this can be achieved by a multi-line command that tells how to run this docker:
```sh
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

- This maps the port 8000 of LXC(401) to the port 8000 of the proxmox host(pve).
- This maps the port 9443 of LXC(401) to the port 9443 of the proxmox host(pve).
- creates a docker container called 'portainer' which will start automatically when docker starts.
- uses 'portainer_data' as the data folder
- is created using the image 'portainer/portainer-ce:latest'

if we are successful, we should be able to open the portainer on a browser:
```sh
root@DockerHost:~# hostname -I
192.168.1.13 172.17.0.1 100.77.8.142 2401:4900:1cb9:92eb:be24:11ff:fe17:**ffe8**
root@DockerHost:~# 
```
so we can use any one of these host names and connect on LAN, as I am using netbird, I will try the URL: `https://100.77.8.142:9443`.

This will warn of risk as there are no SSL certificates, but we can deal with it later. Proceed to the web-page and you get a login interface. This login interface is for portainer and you can setup the admin account and password here.
If you take too long, the portainer instance can also time out :
![034 portainer timeout.png](/img/user/All%20Published%20Notes/Homelab/Images/034%20portainer%20timeout.png)
If this happens, just stop and restart the container:
```sh
root@DockerHost:~# # Check running container 
root@DockerHost:~# docker ps
CONTAINER ID   IMAGE                           COMMAND        CREATED          STATUS          PORTS                                                                                                NAMES
0ddedadb09f8   portainer/portainer-ce:latest   "/portainer"   10 minutes ago   Up 10 minutes   0.0.0.0:8000->8000/tcp, [::]:8000->8000/tcp, 0.0.0.0:9443->9443/tcp, [::]:9443->9443/tcp, 9000/tcp   portainer_container
root@DockerHost:~# 
root@DockerHost:~# 
root@DockerHost:~# docker stop portainer_container
portainer_container
root@DockerHost:~# docker start portainer_container
portainer_container
root@DockerHost:~# 
```

Now you can re-try the login option, if successful, you should get a dashboard with all available containers:
![035 portainer dashboard.png](/img/user/All%20Published%20Notes/Homelab/Images/035%20portainer%20dashboard.png)





---

[^1]:
[^2]:

