---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/009-ssh-into-proxmox-using-netbird/"}
---

created: 2026-05-05
updated: 2026-05-05


To enable SSH from Netbird id (IP or domain name), we need to enable the permission in Netbird dashboard, enable the setting when connecting to Netbird and allow listening on the port where we want to connect.

Steps:
- Allow the host device to listen on the port on which other machines will SSH into
- Connect the host device to Netbird with SSH permission granted
- Allow SSH Access in Netbird dashboard
- Define the port to use for SSH in Netbird Dashboard
- Make the connections


#### Allow the host device to listen on the desired port
In the machine that you want to be able to connect to, update `/etc/ssh/sshd_config` to allow listening on multiple ports. 
The default port is 22, so we want a different port(22022) on which Netbird can connect. 
```sh
root@pve:~# cat /etc/ssh/sshd_config
...
Port 22 # can be used by default connection request
Port 22022 # to be used when connected to netbird
root@pve:~# # restart ssh to make the changes take effect
root@pve:~# sudo systemctl restart ssh 
root@pve:~# 
```
Now, we are listening on both Port 22 and 22022 which we can use for parallel connection:
```sh
root@pve:~# ss -tulpn | grep ssh
tcp   LISTEN 0      128          0.0.0.0:22         0.0.0.0:*    users:(("sshd",pid=30716,fd=8))
tcp   LISTEN 0      128          0.0.0.0:22022      0.0.0.0:*    users:(("sshd",pid=30716,fd=6))
tcp   LISTEN 0      128             [::]:22            [::]:*    users:(("sshd",pid=30716,fd=9))
tcp   LISTEN 0      128             [::]:22022         [::]:*    users:(("sshd",pid=30716,fd=7))
root@pve:~#
```

#### Connect the host device to Netbird with SSH permission
When we start Netbird connection, we need to define what permissions are allowed. This we can do by passing a flag at the time of starting the connection if we are using CLI.
```sh
sudo netbird up --allow-server-ssh
```
This allows SSH connection to be established by making this system an ssh server through Netbird.
##### Key Flags Explained:
- **`--allow-server-ssh`**: Enables the built-in NetBird SSH server. It intercepts traffic on port 22 and routes it securely through the mesh.
- **`--enable-ssh-sftp`**: Critical if you plan to use tools like WinSCP or FileZilla to move files between your Linux Mint PC and Proxmox LXCs.
- **`--enable-ssh-local-port-forwarding`**: Allows you to tunnel services (like a web GUI running on `localhost:8006`) through an SSH connection.
- **`--enable-ssh-root`**: (Optional) If you need to SSH directly as root. Use with caution.

We can check the status and values by seeing the detailed status `netbird status -d` for all devices connected in this netbird or by checking the default JSON file `cat /var/lib/netbird/default.json`.

#### Allow SSH Access in Netbird dashboard
In Netbird dashboard, we need to allow SSH access to a machine. This ensures a machine which does not have SSH access will not be used as a server, ensuring security.

Let's say I want to allow the host machine "Proxmox Host" to be connected to by SSH, I have 2 ways of connecting:
- Connect on port 22, only one connection allowed
- Connect on a custom port, leaving port 22 free for default use.

The common steps include, open the dashboard('https://app.netbird.io/peers') and select the peer machine('Proxmox Host') that you want to allow SSH into. If you have enabled ('up') the netbird with SSH access, you will see Remote Access section enabled, else it will be disabled.
![029 Netbord - SSH access in peer.png](/img/user/All%20Published%20Notes/Homelab/Images/029%20Netbord%20-%20SSH%20access%20in%20peer.png)

We can connect to the machine using the SSH from the same button, but it is also possible to define a rule that allows one group of machines to SSH into another group of machines using policies.

First define the groups(not covering making and adding peers to a group here) that you want to SSH into and the group that can SSH into the other group.

The port to use can be defined in the Policy, but the machine that we want to access should also be listening on the same port for the setting to work.

#### Define the port to use for SSH in Netbird Dashboard
In the left side of the dashboard, select *Access Control -> Policies*('https://app.netbird.io/access-control'), and add a new policy for SSH access:
![030 Netbird New Policy.png](/img/user/All%20Published%20Notes/Homelab/Images/030%20Netbird%20New%20Policy.png)
1. The protocol to use, 
	1. Use 'TCP' if you want to use SSH with a fixed port
	2. Use 'Netbird SSH' if you want to use port 22 to connect using the default set up in netbird
2. The source group who are allowed to SSH into the destination
3. The destination group who can be SSH-ed into
4. The direction of access, the default is bi-directional (both source and destination can access each other).
5. Ports: you can use this to define the port that you want to use for access ('Netbird SSH' does not allow editing the port, it uses 22 internally).
Once you define the rules, add "posture checks"(Optional) and add a name and save.

We can allow SSH access in the default port 22, but that blocks parallel connection to the machine with both IP address and Netbird IP. If we want to ensure we can use SSH with both actual IP and Netbird IP, we need to use TCP policy so that the actual IP is available to connect as needed.

I defined the protocol as TCP and the port was same as defined in "[[#Allow the host device to listen on the desired port|Listen on desired port]]" section (22022).
![031 Netbird SSH policy.png](/img/user/All%20Published%20Notes/Homelab/Images/031%20Netbird%20SSH%20policy.png)


#### Make the connections

Now we can use the default port(22) for any SSH request which is not from Netbird(`ssh root@192.168.1.101 -p 22`), or we can connect using Netbird IP or device domain name(`ssh root@100.77.214.80 -p 22022` or`ssh root@proxmox-host.netbird.cloud -p 22022`). This allows me to connect to the System even if the IP address changes due to something like Router change.



---

[^1]:
[^2]:

