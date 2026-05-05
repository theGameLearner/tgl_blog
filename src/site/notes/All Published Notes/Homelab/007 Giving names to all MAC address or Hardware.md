---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/007-giving-names-to-all-mac-address-or-hardware/"}
---

created: 2026-04-04
updated: 2026-05-05

As IP address and port will become hard to manage, I want a service where I can write the IP and port once and use a custom name for all use cases. Like a local DNS(Domain Name service) for my LAN.

I checked a few options to identify the best option for me. The idea is the option I choose should be scalable with time, allow all of my devices (with multiple emails) as well as my wife's account to log in for free as I am at the starting point and don't want to pay for services that I may rarely use.

DNS and VPN options:
- [Technitium](https://technitium.com/dns/): for people with good understanding and knowledge who need more control and have atleast medium level of understanding of how to manage a DNS
	- I did not find it easy to start with
- [Pi-hole](https://pi-hole.net/): Easy solution for hobbyist to use directly without deep dive.
	- It did not feel as a ready to use solution
- [adguard-home](https://adguard.com/en/adguard-home/overview.html): seems to be great option for ad blocking
	- My main focus is going to be accessing remote machine using the SSH and "RustDesk".
- [TailScale](https://tailscale.com/): A great choice and the first option for free reliable VPN
- [Netbird](https://netbird.io/): Can serve as DNS as well as VPN and can also be self-hosted. This is a better alternative to TailScale as well as I have 5 email address in my family. 

I used TailScale and the only reason to switch to / choose NetBird is that I need more user accounts(5) vs the TailScale free tier(3).

# Netbird on the HomeLab
## LXC to use NetBird from
The LXC I use for Netbird will be 'unprivileged' as I am not sure about the security aspect and want to be sure of its working before everyone at home starts to use it.

#### Create the LXC
step one is to create this LXC, let's say proxmox id 401 using the steps that we used to make NAS LXC. ([[All Published Notes/Homelab/003 Proxmox - Ubuntu LXC as NAS with personal HDDs#Create a new CT\|create a new LXC Container]])

![025 NetBird Main LXC.png](/img/user/All%20Published%20Notes/Homelab/Images/025%20NetBird%20Main%20LXC.png)

This is the created LXC's config file:
```bash
root@pve:~# cat /etc/pve/lxc/401.conf
arch: amd64
cores: 2
features: nesting=1
hostname: NetBird-LXC
memory: 2048
net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=BC:24:11:14:EB:10,ip=192.168.1.201/24,ip6=dhcp,type=veth
ostype: ubuntu
rootfs: local-lvm:vm-401-disk-0,size=2G
swap: 1024
unprivileged: 1
root@pve:~# 
```

#### Allow the LXC access to crate and update virtual network interface
add the following lines to '/etc/pve/lxc/401.conf':
```sh
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

understanding:
- `lxc.cgroup2.devices.allow: c 10:200 rwm` - Grants the container access to a specific **character device** on the host with Major as `10` (device class "misc") and Minor as `200` (specifically `/dev/net/tun`: the TUN/TAP virtual network interface device)
	- `rwm` - read, write and mknod (create device nodes).
- `lxc.mount.entry: /dev/net dev/net none bind,create=dir` - Binds the host’s `/dev/net` directory into the container at the same path (`/dev/net`).
- `lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file` - Bind‑mounts the actual TUN device file from the host (`/dev/net/tun`) into the container at the same location.

```bash
root@pve:~# cat /etc/pve/lxc/401.conf
arch: amd64
cores: 2
features: nesting=1
hostname: NetBird-LXC
memory: 2048
net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=BC:24:11:14:EB:10,ip=192.168.1.201/24,ip6=dhcp,type=veth
ostype: ubuntu
rootfs: local-lvm:vm-401-disk-0,size=2G
swap: 1024
unprivileged: 1
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
root@pve:~# 
```

Restart the container:
```sh
# stop and start if the container was running
pct stop 401 && pct start 401
# alternative Reboot coomand
pct reboot 401
# OR
# start the container if already stopped
pct start 401
```

*Verify* inside the container that `/dev/net/tun` exists and is usable:
```bash
root@pve:~# pct enter 401
root@NetBird-LXC:~# ls -l /dev/net/tun
crw-rw-rw- 1 nobody nogroup 10, 200 Apr 15 17:02 /dev/net/tun
root@NetBird-LXC:~#
```

Now, we can start using this LXC as a NetBird endpoint.
You can run 'update' and 'upgrade' as optional commands as the system is a fresh install.

> [!Note]
> validate that TUN is available on the host (Proxmox) by `lsmod | grep tun` or `cat /boot/config-$(uname -r) | grep CONFIG_TUN` before you proceed.

> [!Warning]
> I forgot to set the Gateway to `192.168.1.1` which blocked network access and then I fixed it later, check if you updated the correct gateway on your end.

## Install NetBird on the LXC machine

Follow the installation guide from Netbird [documentation](https://docs.netbird.io/get-started/install/linux) to download and install netbird on the machine if the article is outdated.

Before we use the default installation script, we need `curl`, .
Install these by the line:
```sh
apt install curl

```

run the script to install netbird on the LXC by typing the command in the LXC's terminal as root user:
```sh
# Run script to setup and run NetBird on a linux machine.
curl -fsSL https://pkgs.netbird.io/install.sh | sh
```

Check Netbird is running:
```bash
netbird status
```

Now we will start netbird and set this LXC as the exit node:
```sh
netbird up
```

you get a URL that you can use in any browser to verify with an account. I am using my GMail id 'thegamelearner@gmail.com' as Google is a secure authenticator.
```sh
root@NetBird-LXC:~# netbird up            
Please do the SSO login in your browser. 
If your browser didn't open automatically, use this URL to log in:

https://login.netbird.io/activate?user_code=XXXX-XXXX


Alternatively, you may want to use a setup key, see:

https://docs.netbird.io/how-to/register-machines-using-setup-keys
Connected
root@NetBird-LXC:~# 
```

The `Connected` line only appears after user has authorized in 'https://login.netbird.io/activate' URL. The code is a Single use and keeps changing for every user.

The status should also change now:
```sh
root@NetBird-LXC:~# netbird status
OS: linux/amd64
Daemon version: 0.68.3
CLI version: 0.68.3
Profile: default
Management: Connected
Signal: Connected
Relays: 4/4 Available
Nameservers: 0/0 Available
FQDN: netbird-lxc.netbird.cloud
NetBird IP: 100.77.84.243/16
Interface type: Kernel
Quantum resistance: false
Lazy connection: false
SSH Server: Disabled
Networks: -
Peers count: 0/0 Connected
root@NetBird-LXC:~# 
```

## Install NetBird on other machines 
### Android setup
Using the play store, I will now install NetBird on my Android phone as a way to test the connection when both are on different internet (WiFi vs cellular).

I installed the [play store app for android](https://play.google.com/store/apps/details?id=io.netbird.client&hl=en) and clicked on connect. As I use the same email, the connection happened easily.

### Chromebook setup
As I do not want to use the same email but want to log in to NetBird on my Chromebook, I will use a setup Key. A Setup Key allows a device to join your network without needing an interactive browser login. It "pre-authorizes" the device under your account.

In the left panel, select "Setup Keys" and add it. Set expiry and other details as you want to.


Chromebooks are a bit unique because you have **two** ways to run NetBird:
- Option A: The Android App (Easiest)
	- Install the NetBird app from the Google Play Store on your Chromebook.
	- Open the app and go to Settings (usually a gear icon or three dots).
	- Look for an option to Enter Setup Key.
		- In left side, select Change Server, set the URL for Server as netbird cloud server url (if using cloud hosting instead of self hosting) : `https://api.netbird.io:443`
			- Below server url, use "+ Add this device with a setup key" and add the server key.
	- Paste your key and hit Connect. 
	- The Chromebook will instantly appear in your list of peers without asking for an email login.
- Option B: The Linux (Crostini) Way (Best for Devs)
	- you might prefer running it inside the Linux environment of the Chromebook:
		- Open the Terminal app on your Chromebook.
			- Install NetBird using the same command you used for the LXC: `curl -fsSL https://pkgs.netbird.io/install.sh | sh`
			- Connect using the setup key directly: `sudo netbird up --setup-key YOUR-KEY-HERE`

### Proxmox setup with SSH
Proxmox uses Linux, so setup is similar to Linux terminal setup. 
To enable SSH login, we need to allow the SSH control in Netbird settings.
in terminal, disconnect the NetBird vpn, and re-enable with SSH permission:
```sh
root@pve:~# netbird down
Disconnected
root@pve:~# netbird up --allow-server-ssh --enable-ssh-root
Connected
root@pve:~# 
```
Understanding:
- `--allow-server-ssh`: Turns on the NetBird-managed SSH server.
- `--enable-ssh-root`: Specifically allows the `root` user to log in.

Now, we also need to allow this in NetBird Dashboard, so we can cover this in next section.

#### Settings in Dashboard
Now, in a browser, I created groups for future use and also set the LXC as the exit node.

I also have disabled "Session Expiry" for the LXC as it is the main machine I want to use for my system. 

Enable SSH for the machine you want to control using SSH, in my case, Proxmox Host and Linux Mint(Main Machine):
![026 Allowing SSH in Dashboard.png](/img/user/All%20Published%20Notes/Homelab/Images/026%20Allowing%20SSH%20in%20Dashboard.png)

This also needs NetBird SSH access policy to be created to define which group's users can access which machine using SSH
![027 Netbird Creating SSH policy.png](/img/user/All%20Published%20Notes/Homelab/Images/027%20Netbird%20Creating%20SSH%20policy.png)

If it is done properly, we can use NetBird to SSH into a machine without it's IP address:
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ netbird ssh root@proxmox-host.netbird.cloud
SSH authentication required.
Please do the SSO login in your browser.
If your browser didn't open automatically, use this URL to log in:

https://login.netbird.io/activate?login_hint=thegamelearner%40gmail.com&user_code=VZQR-FWFW
Or visit: https://login.netbird.io/activate?login_hint=thegamelearner%40gmail.com and enter code: VZQR-FWFW

Waiting for authentication...
Authentication successful!
root@pve:~# pwd
/root
root@pve:~# ip route
default via 192.168.1.1 dev vmbr0 proto kernel onlink 
100.77.0.0/16 dev wt0 proto kernel scope link src 100.77.214.80 
192.168.1.0/24 dev vmbr0 proto kernel scope link src 192.168.1.101 
192.168.1.0/24 dev wlp41s0 proto kernel scope link src 192.168.1.9 
root@pve:~# pct enter 401

root@NetBird-LXC:~# ip route
default via 192.168.1.1 dev eth0 proto static 
100.77.0.0/16 dev wt0 proto kernel scope link src 100.77.84.243 
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.201 
root@NetBird-LXC:~# exit 
exit
root@pve:~# exit
logout
thegamelearner@thegamelearner-MS-7E12 ~ $ ssh root@proxmox-host.netbird.cloud
SSH authentication required.
Please do the SSO login in your browser.
If your browser didn't open automatically, use this URL to log in:

https://login.netbird.io/activate?login_hint=thegamelearner%40gmail.com&user_code=FDZF-KNHF
Or visit: https://login.netbird.io/activate?login_hint=thegamelearner%40gmail.com and enter code: FDZF-KNHF

Waiting for authentication...
root@pve:~# pwd
/root
root@pve:~# ip route
default via 192.168.1.1 dev vmbr0 proto kernel onlink 
100.77.0.0/16 dev wt0 proto kernel scope link src 100.77.214.80 
192.168.1.0/24 dev vmbr0 proto kernel scope link src 192.168.1.101 
192.168.1.0/24 dev wlp41s0 proto kernel scope link src 192.168.1.9 
root@pve:~# pct enter 401
root@NetBird-LXC:~# ip route
default via 192.168.1.1 dev eth0 proto static 
100.77.0.0/16 dev wt0 proto kernel scope link src 100.77.84.243 
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.201 
root@NetBird-LXC:~# exit
exit
root@pve:~# exit
logout
thegamelearner@thegamelearner-MS-7E12 ~ $ 
```


## Future steps 
### LXC as exit node
We can set a node(any computer or LXC) as a exit node. Meaning, all my devices will send their online signals through netbird to this one node, this one node alone will talk to the internet, and will be my only source of internet.
Benefits:
- Security when using public WiFi in places like airport, as the wifi only sees data communicated to your exit node and not what actually is being seen or visited.
- Bypass Local DNS Hijacking: It prevents the public Wi-Fi from redirecting your DNS queries or injecting ads into your browsing session.
- Access content as if you are where your exit node is.
- Centralized Ad-Blocking and Filtering: By setting that server as your exit node, your mobile devices (phones/tablets) gain the benefits of your home's DNS-based ad-blocking even when you are on 5G or LTE. This saves data and improves battery life by preventing ads from loading.

To set it up, select the node in netbird dashboard and add it as a exit node. Remember that the exit node needs to be 'online' to access internet if you set an exit node and are connected to netbird, we also need a public nameserver (google or Cloudflare) if we want to access anything outside the netbird mesh.
### Create the Exit Node in the Dashboard

1. Open your browser and go to the [NetBird Network Routes](https://app.netbird.io/network-routes) page.
2. Click **Add Route**.
3. Fill in the details exactly like this:
    - **Network Identifier:** `Home-Internet-Exit`
    - **Description:** `LXC Exit Node for Family`
    - **Network Range:** `0.0.0.0/0` (This signifies "All Internet Traffic")
    - **Routing Peer:** Select your **NetBird-LXC**.
    - **Masquerade:** **Enabled (On)** — _Crucial: This allows the LXC to act as a gateway._
    - **Distribution Groups:** Select **All** (or a specific group if you only want your devices to use it).
4. Click **Add Route**.

This did not work after restart of my LXC, not sure why, will explore in future again.
#incomplete 

### Use Custom(Cloudflare) DNS From LXC for all connected devices
How can I use Cloudflare DNS so that I am not locally restricted by my ISP's DNS system.
In the Netbird dashboard, on the left side, open "DNS -> Nameservers" and add the Nameserver you want to use, including any custom DNS options. 
This along with the distribution groups will allow the group users to use this DNS as needed.
![028 Netbird DNS settings.png](/img/user/All%20Published%20Notes/Homelab/Images/028%20Netbird%20DNS%20settings.png)

Now in Peers section, we can rename all machines connected, and use them instead of remembering IP addresses for each machine. This serves the need for a DNS for all machine that will not change even if the IP address changes.


---

[^1]: https://www.youtube.com/watch?v=S5D6uksel10
[^2]: https://docs.netbird.io/get-started/install/linux


