---
{"dg-publish":true,"permalink":"/all-published-notes/computer/operating-system/chrome-os/creating-ssh-server-in-chromebook/"}
---

created: 2025-07-26
updated: 2025-07-29

### Defining the username and password:
- We need a account password, if it was not set, the authentication from the machines can fail
- \[ChromeOS\] \[Terminal\] Change to superuser, as it can set other user's password: `sudo su`
- \[ChromeOs\] \[Terminal\] then enter `passwd` to change the root password, or use `passwd thegamelearner` to set password for the user 'thegamelearner', after setting the password, you can exit or stay here as a root user
- \[ChromeOs\] \[Terminal\] Let us assume the password is : 'GameLearning$#22'

---
### Check the status of openssh-server
- \[ChromeOs\] \[Terminal\] check the current system status by starting the server: `sudo service ssh start`
	- If this fails because of `conditionPathExists=~/etc/ssh/sshd_not_to_be_run was not met`, we can either delete this file, ==or==
	- we can purge the 'openssh-server' and then install it again: `sudo apt purge openssh-server && sudo apt install openssh-server`
- \[ChromeOs\] \[Terminal\] stop the server if it was started, as default options use port 22, which is not a great option: `sudo service ssh stop`

---
### Port Forwarding on desired port
- \[ChromeOs\] \[Settings\] open settings, in linux port forwarding, add/enable the port(between 1024 and 65535) you want to use for ssh
- \[ChromeOs\] \[Settings\] I am keeping both TCP and UDP active as I am not sure which to use, in my case port is: '1234'
- \[ChromeOs\] \[Terminal\] Modify the SSH config file at `/etc/ssh/sshd_config` using something like vim or nano, I used micro with sudo permission: `sudo micro /etc/ssh/sshd_config`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] add the following lines:
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `Port 1234`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `AllowUsers itrishabhjain`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `PermitRootLogin no`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `ListenAddress 0.0.0.0`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `PasswordAuthentication yes`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `AllowAgentForwarding yes`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `AllowTcpForwarding yes`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] save the file and exit : 'ctrl+s' and then 'ctrl+q'
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] only add the texts inside the quote, eg., for `Port 1234`, type 'Port 1234' in a single line without the quotes
- \[ChromeOs\] \[Terminal\] verify the edited file is correct in syntax : `sudo sshd -t`
- \[ChromeOs\] \[Terminal\] the new port needs to be set in '/etc/systemd/system/ssh.socket.d/override.conf' as well, because sometimes ChromeOs does not allow us to use new port for ssh
- \[ChromeOs\] \[Terminal\] edit the socket file by `sudo systemctl edit ssh.socket` which will edit '/etc/systemd/system/ssh.socket.d/override.conf' in the default editor
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] add the following lines:
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] `[Socket]`
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] `ListenStream=`
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] `ListenStream=1234`
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] save the file and exit : 'ctrl+s' and then 'ctrl+q'
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] 'ListenStream=' is important as it removes any old port being used.
- \[ChromeOs\] \[Terminal\] reload the systemd daemon : `sudo systemctl daemon-reload` which will use updated configurations meaning the new port you defined.

---
### start ssh service
- \[ChromeOs\] \[Terminal\] Restart the SSH Services: `sudo systemctl restart ssh.socket` and `sudo systemctl restart ssh.service` or you can use `sudo service ssh start`
- \[ChromeOs\] \[Terminal\] verify the port has changed: `sudo service ssh status` should return the new port in use.
- \[ChromeOs\] open wifi setting and get the ip address of the system, click on wifi, go into wifi connected, use the 'i' button to see ip address, e.g., '192.168.31.48'. Most of the times, it will start with '192.168.'

---
### Connect to the created SSH
- \[linux\] \[Terminal\] open terminal in linux machine, use the ssh command to tunnel into chromeOS `ssh -p 1234 thegamelearner@192.168.31.48`
- \[linux\] \[Terminal\] if asked 'The authenticity of host '\[192.168.31.48\]:1234 (\[192.168.31.48\]:1234)' can't be established.', 'Are you sure you want to continue connecting (yes/no/\[fingerprint\])?' type `yes`
- \[linux\] \[Terminal\] prompted for password, type the password set in ChromeOS: GameLearning$#22

>Remember that there is often no user feedback, so you need to type the password and hit 'enter' even when you cannot see it being typed.

---
### Use hostname for chromebook Connection
When we are connecting to same chromebook multiple times, the IP address of chromebook can change, this will lead to issues in SSH authentication or adding new hosts for every connection, leading to lot of efforts.
We can assign or use a hostname for our device, which will be auto converted to our IP address by the router.

#### Changing hostname (old method)
- \[ChromeOs\] \[google chrome\] in google chrome, go to url: [chrome://flags](chrome://flags)
- \[ChromeOs\] \[google chrome\] In the new window, search 'hostname' as you want to allow yourself to set a hostname for the device
	- Change this setting to "*Enabled*"
	- ==Restart== the system after changing the setting.
- \[ChromeOs\] \[Settings\] go to "About ChromeOs" -> "Additional details" -> "Device Name" and now you can use or change this name.

#### Changing hostname (new method)
- \[ChromeOs\] \[Terminal\] type `hostname` to see the hostname, or `hostnamectl` to see all details, let's say it is 'Ridhi'
- \[ChromeOs\] \[Terminal\] we can use `sudo hostnamectl set-hostname <new host name>` to set new host name, e.g., `sudo hostnamectl set-hostname Ridhi`
- \[ChromeOs\] \[Terminal\] edit the file in '/etc/hosts', replace the host name of '127.0.1.1' from old hostname to the new one, e.g., `127.0.1.1    Ridhi`

#### Allowing your system to use hostname instead of ip
We have added a hostname to the chromebook, but it is not going to work if your own system does not know how to use this hostname to find the IP address at runtime.
We have multiple solutions for DNS(domain name system) which will allow your system to connect to other systems on Local area network. For ease of use, I will use *'Avahi-daemon'* and *'nss-mdns'*.

[Avahi](http://avahi.org/):*Avahi-daemon* is a service that provides network service discovery using mDNS (Multicast DNS) and DNS-SD (DNS Service Discovery) protocols. It allows devices and applications to find each other on a local network without needing to configure the DNS server manually.

[`nss-mdns`](https://github.com/avahi/nss-mdns) is a plugin for the GNU Name Service Switch (NSS) functionality of the GNU C Library ('glibc') providing host name resolution via [Multicast DNS](http://www.multicastdns.org/) (aka _Zeroconf_, aka _Apple Rendezvous_, aka _Apple Bonjour_), effectively allowing name resolution by common Unix/Linux programs in the ad-hoc mDNS domain '.local'. 'nss-mdns' provides client functionality only, which means that you have to run a mDNS responder daemon seperately from 'nss-mdns' if you want to register the local host name via mDNS.

##### Install Avahi and NSS-mDNS if necessary
###### Debian based systems (Ubuntu, Linux mint, etc.)
For Debian based machine, the packages are called 'avahi-daemon' and 'libnss-mdns' which can be installed by:
```bash title:debian
sudo apt install avahi-daemon libnss-mdns
```
If you are using chromebook or ChromeOs, you also need 'avahi-utils':
```bash title:Crostini
sudo apt install avahi-daemon avahi-utils libnss-mdns
```

###### Fedora/RHEL-based systems
For fedora, we need to install and enable them:
```bash
sudo dnf install avahi nss-mdns
sudo systemctl enable --now avahi-daemon.service
```

###### Arch based systems
For Arch based system, we also need to install and enable them:
```bash
sudo pacman -S avahi nss-mdns
sudo systemctl enable --now avahi-daemon.service avahi-daemon.socket
```

##### Configure Name Service Switch ('/etc/nsswitch.conf')
After installing 'libnss-mdns', its post-installation script *should* modify '/etc/nsswitch.conf' to include 'mdns4_minimal' (or 'mdns') in the 'hosts:' line. This tells your system to consult mDNS for hostname resolution _before_ trying traditional DNS. This process is fast and will not slow your internet unless you are configuring it on a massive scale.

Open '/etc/nsswitch.conf' on your Linux machine: `sudo micro /etc/nsswitch.conf`
Look for the line that starts with '==hosts:=='. It should look something like this (the exact order can vary, but 'mdns4_minimal' should be present and preferably _before_ 'dns'):

```
hosts: files mdns4_minimal [NOTFOUND=return] dns
```

##### Finding the networking service in your distro
There are many types of network managers or service that can handle internet access in a linux distro. Based on your distribution, you can check it's status and restart it as needed.
An option is to search all services for 'network' text: `systemctl list-unit-files --type=service --state=enabled | grep network`.
Alternatively, the easiest way to find your network service is to check their status, any option which is on will provide results.
```bash
# search all enabled services to find 'network' in the service name
systemctl list-unit-files --type=service --state=enabled | grep network

systemctl status networking # did not work on Linux Mint
# Unit networking.service could not be found.

systemctl status NetworkManager # worked on Linux Mint, service is 'enabled' and status is 'active'
# Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; enabled; preset: enabled)
# Active: active (running) since Tue 2025-07-29 11:26:34 IST; 3h 41min ago

systemctl status systemd-networkd # did not work on Linux Mint, service is 'disabled' and is in 'dead' status
# ○ systemd-networkd.service - Network Configuration
#     Loaded: loaded (/usr/lib/systemd/system/systemd-networkd.service; disabled; preset: enabled)
#     Active: inactive (dead)

systemctl status systemd-resolved # worked on Linux Mint, service is 'enabled', and status is 'active'
#● systemd-resolved.service - Network Name Resolution
#     Loaded: loaded (/usr/lib/systemd/system/systemd-resolved.service; enabled; preset: enabled)
#     Active: active (running) since Tue 2025-07-29 11:26:32 IST; 3h 41min ago
```
##### using updated configurations
Restart the daemon service for avahi: 
```bash
# restart the daemon service
sudo systemctl restart avahi-daemon.service 
# restart networking or NetworkManager, systemd-resolved, etc., depending on your distro
sudo systemctl restart NetworkManager.service
OR
sudo service network-manager restart
```
Network Manager is a safer option for us, as it might be controlling 'systemd-resolved' service.
Sometimes this may not work, so we can simply *restart the system*.

##### Verify mDNS resolution:
do this to check whether the current IP can be found: `avahi-resolve-host-name Ridhi.local`
or ping the server: `ping Ridhi.local`
if these work, you can ssh to the server: `ssh -p 1234 thegamelearner@Ridhi.local`

---
### What if it fails?
###### Connectivity fails due to something blocking the conversation
In my case, even after re-starting both machines, the setup did not work.
you can use `avahi-browse -art` to see all other connected systems that have mDNS/DNS-SD services using the Avahi daemon. 
If it does not show the other device's 'ip address'or the name that you set, it might not be published as a host. 
**To check connectivity**, 
- from ChromeOs to Linux: 
	- in the device which we wanted to host(ChromeOs), we will install avahi and try to *publish* a test connection: `avahi-publish-service "ChromebookTestService" _ssh._tcp 1235 "Chromebook Test" &`
	- Now, in our Linux machine, try `avahi-browse -art` to find all service at port 1235, 
		- if this has *no result* at this port, your chromeOs is unable to send information to linux machine. This can be because of firewall or other setup issues in router.
		- if you find your ChromeBook's test connection but `avahi-resolve-host-name Ridhi.local` command had failed. Then we can confirm that one-way communication is established. There is a chance that chromebook's host name and address are not exposed properly.
```bash hl:2,7
...
= enp7s0 IPv4 ChromebookTestService                         SSH Remote Terminal  local
   hostname = [Ridhi.local]
   address = [192.168.31.49]
   port = [1235]
   txt = ["Chromebook Test"]
= wlp14s0 IPv4 ChromebookTestService                         SSH Remote Terminal  local
   hostname = [Ridhi.local]
   address = [192.168.31.49]
   port = [1235]
   txt = ["Chromebook Test"]
...
```


- from Linux to ChromeOs:
	- we will have to *publish* test connection from linux to check if chromebook's linux terminal can find it: `avahi-publish-service "TestMachine" _ssh._tcp 22 "Test SSH Service" &`
	- Now, in chromeOs, we can use `avahi-browse -art` to search this connection
		- if the connection is *not found*, there is a problem with setup or firewall
		- if the connection is found, you can be sure, that both machines can talk, then we just need to resolve the connection issue.
```
=   eth0 IPv6 TestMachine                                   SSH Remote Terminal  local
   hostname = [thegamelearner-MS-7E12.local]
   address = [0000:ffff:000:ffff:ffff:ffff:ffff:ffff]
   port = [1234]
   txt = ["Test SSH Service"]
```

###### Linux machine's Name Service Switch (NSS) configuration not using Avahi for resolution
The problem now boils down to your Linux machine's Name Service Switch (*NSS*) configuration **not using Avahi for resolution** when you use *ping* or *avahi-resolve-host-name* (which uses NSS to some extent for resolution logic).
###### Checking by disabling firewall
check firewall status: `sudo ufw status`
disable firewall: `sudo ufw disable`
if you use 'firewalld', check it's status `sudo systemctl status firewalld` and stop if it is in use `sudo systemctl stop firewalld`.
Check if we can access it now: `ssh -p 1234 thegamelearner@Ridhi.local`
*if we can access it now*, enable the port in firewall: `sudo ufw allow 1234/tcp` then re-enable the firewall `sudo ufw enable`

if it still does not work, All the best to figure it out 👍