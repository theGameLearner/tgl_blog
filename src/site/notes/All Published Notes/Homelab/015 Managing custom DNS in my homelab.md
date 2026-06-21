---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/015-managing-custom-dns-in-my-homelab/"}
---

created: 2026-06-11
updated: 2026-06-16

#### General
A common problem with Homelab is that I have already made 2 LXC('NAS-LXC' and 'DockerHost') with users, groups, ports and other settings that are becoming harder and harder to track without a dedicated file to track all of it.

Now, I can write a file and store the ports, user name, group, permission or comments about the data, etc., but, what about the domains? I cannot open a note file to search each domain that I want to access just to avoid learning all the IP addresses attached to each domain.

But browsers talk to internet and uses lot of information but all I remember is to type 'google.com'.

This is made possible by **DNS**(Domain Name Server), which is a interesting topic if you want to deep dive into. In most simple explanation: When we type a name in browser, the browser uses DNS to resolve the name, if it is able to resolve it, it will open the webpage, if it can't, it will give errors.

#### Goal
Now I want to create a DNS record inside my home, so my own services will remain available to me without having to write the IP address and port. This has 2 parts:
- A DNS server that listens on port `53` (standard) and either forwards the request to another DNS or catches the request and forward it to my homelab ip to be processed (coreDns)
- A file which defines which url goes to which ip and port (caddy)


> [!Warning]
> This container setup will break if we change router as the IP addresses are integral part of the setup.
> See: [[All Published Notes/Homelab/005 Impact of Changing ISP or Routers with Proxmox based Homelab\|Impact of changing router]]


### Define the files and folders
#### Create folders to hold files and data

Before starting, I mounted my "homelab/apps" folder to "/apps" folder inside LXC, this does not have any impact on next steps, but still mentioning here:
```sh
root@pve:~# pct config 401 | grep mp
mp0: /mnt/main-backup/homelab_backups/vikunja_lxc_backup,mp=/vikunja_backup
mp1: /mnt/main-backup/homelab/apps,mp=/apps
root@pve:~#
```

The first thing I need is a folder where all my files will reside, I will use 'localDns' inside '/apps' for this. Inside 'localDns', I want two folder, 'caddy' and 'coredns' which will then have data and config folders. 
```sh
root@DockerHost:~# find /apps/localDns -maxdepth 2 -type d
/apps/localDns
/apps/localDns/caddy
/apps/localDns/caddy/config
/apps/localDns/caddy/data
/apps/localDns/coredns
root@DockerHost:~# 
```

folder details:
- */apps/localDns*: main folder where all files and data will stay
- */apps/localDns/caddy*: has caddy config files and data files
- */apps/localDns/caddy/config*: config files for caddy
	- defines the actual ip and port to use and the name of the service
- */apps/localDns/caddy/data*: files created inside container
	- stores TLS certificates, OCSP stapling responses, etc
- */apps/localDns/coredns*: defines the rules for the DNS.

#### Define the Core DNS config file
Create a `Corefile` which will have config for coreDNS in '/apps/localDns/coredns/Corefile'.

We can use '/etc/coredns/hosts' file to define each url and the IP address (LXC's IP) where the request will be sent for further processing. 

> [!Warning]
> As I do not want to add new services manually in coreDNS as well as caddy file, I will write a regular expression and any and all urls in the format of 'abc.tglservice'(`tglservice`) will be forwarded to my LXC ip. This fails as your local machine may not find the homelab to give reply as fast as the backup cloudflare server(1.1.1.1), so using a hosts file is a better option.

As I am using this for all communication, we will use port 53 to define redirection to the homelab. The other requests will be forwarded to `1.1.1.1`(cloudflare) and `9.9.9.9`(quad9) to be managed by those DNS.

the created file(`/apps/localDns/coredns/Corefile`):
```sh
root@DockerHost:~# ls -al  /apps/localDns/coredns/Corefile 
ls: cannot access '/apps/localDns/coredns/Corefile': No such file or directory
root@DockerHost:~# fresh /apps/localDns/coredns/Corefile
root@DockerHost:~# cat /apps/localDns/coredns/Corefile
.:53 {
    log
    errors

    # Use hosts file for exact tglservice names (no regex needed)
    hosts {
        192.168.1.100 portainer.tglservice
        192.168.1.100 vikunja.tglservice
        192.168.1.100 browser.tglservice
        192.168.1.100 health.tglservice
        # Optional: catch-all for any other *.tglservice (requires wildcard, but hosts doesn't support it)
        # We'll handle wildcard with a separate block or rewrite – but you can list all needed subdomains.
        fallthrough
    }

    # Forward all other domains (including unknown .tglservice) to upstream
    forward . 1.1.1.1 9.9.9.9 {
        max_concurrent 1000
        prefer_udp
    }

    cache 3600
}
root@DockerHost:~# 
```

#### Define the Caddy file which has ip and ports

Now, we want to define the caddy file which will redirect the request to correct IP and port. To do this, we make a file called 'Caddyfile' in `/apps/localDns/caddy/config/Caddyfile`:
```
root@DockerHost:~# cat /apps/localDns/caddy/config/Caddyfile
# =====================================================================
# GLOBAL OPTIONS BLOCK
# =====================================================================
# Settings placed here apply to the entire Caddy server globally.
{
    # Disable automatic HTTPS for local domains (stops Caddy from trying 
    # to obtain public Let's Encrypt certificates for internal URLs)
    auto_https off

    # --- ADVANCED GLOBAL OPTION EXAMPLES (Commented for future reference) ---
    #
    # 1. Change the admin interface bind address or disable it completely:
    # admin off
    # admin 127.0.0.1:2019
    #
    # 2. Set default HTTP/HTTPS ports if you are customizing host binds:
    # http_port 80
    # https_port 443
    #
    # 3. Configure global logging formats or send files to specific paths:
    # log {
    #     output file /var/log/caddy/access.log {
    #         roll_size 10mb
    #         roll_keep 5
    #     }
    # }
    #
    # 4. Adjust global timeout parameters for connections:
    # servers {
    #     timeouts {
    #         read_body   10s
    #         read_header 10s
    #         write       10s
    #         idle        2m
    #     }
    # }
}

# =====================================================================
# REVERSE PROXY & HOST DEFINITIONS
# =====================================================================

# 1. Vikunja Task Management Service
http://vikunja.tglservice {
    # Send access logs directly to stdout (visible in Docker/Portainer logs)
    log {
        output stdout
    }

    reverse_proxy {
        # Upstream targets: Tries physical LAN first, falls back to Netbird overlay
        to 192.168.1.100:3456 100.77.8.142:3456
        
        # Priority failover rules: always use the 1st IP unless it goes offline
        lb_policy first
        fail_duration 5s
        max_fails 1
    }
}

# 2. Portainer Management Dashboard
http://portainer.tglservice {
    log {
        output stdout
    }

    reverse_proxy {
        # Upstream targets (Portainer communicates natively via internal HTTPS)
        to https://192.168.1.100:9443 https://100.77.8.142:9443
        
        # Configure reverse proxy network layers to skip self-signed SSL errors
        transport http {
            tls_insecure_skip_verify
        }
        
        lb_policy first
        fail_duration 5s
        max_fails 1
    }
}

# 3. Isolated Dedicated Browser Container
https://browser.tglservice {
    
    # tls internal # creates an internal certificate for secure access
    
    log {
        output stdout
    }

    reverse_proxy {
        # Upstream targets
        to https://192.168.1.100:10001 https://100.77.8.142:10001
     
        # Configure reverse proxy network layers to skip self-signed SSL errors
        transport http {
            tls_insecure_skip_verify
        }
           
        lb_policy first
        fail_duration 5s
        max_fails 1
    }
}


# =====================================================================
# UTILITY AND FALLBACK BLOCKS
# =====================================================================

# 4. Global Health Checker 
# Resolves cleanly at http://health.tglservice
http://health.tglservice {
    log {
        output stdout
    }
    respond "OK" 200
}

# 5. Catch-All Default 404 Error Block
# If any device hits port 80 of your proxy using a non-existent sub-domain 
# (e.g. typosquatting or invalid .tglservice string), drop it safely with a 404.
http://*.tglservice, http://tglservice {
    log {
        output stdout
    }
    respond "Service Not Found" 404 {
        close
    }
}

```


> **Explanation:**  
> - `auto_https off` prevents Caddy from trying to obtain Let's Encrypt certificates for `.tglservice` domains.  
> - `tls_insecure_skip_verify` is used because Portainer and the browser container use self‑signed HTTPS certificates.

Now the files are:
```sh
root@DockerHost:~# find /apps/localDns -maxdepth 3 -printf "%y %p\n"
d /apps/localDns
d /apps/localDns/caddy
d /apps/localDns/caddy/config
f /apps/localDns/caddy/config/Caddyfile
d /apps/localDns/caddy/data
d /apps/localDns/coredns
f /apps/localDns/coredns/Corefile
root@DockerHost:~# find /apps/localDns -maxdepth 3 -ls
      163    128 drwxrwxrwx   4 nobody   nogroup    131072 Jun 13 11:57 /apps/localDns
      164    128 drwxrwxrwx   4 nobody   nogroup    131072 Jun 13 12:19 /apps/localDns/caddy
      166    128 drwxrwxrwx   2 nobody   nogroup    131072 Jun 13 13:55 /apps/localDns/caddy/config
      169    128 -rwxrwxrwx   1 nobody   nogroup      3445 Jun 13 13:55 /apps/localDns/caddy/config/Caddyfile
      167    128 drwxrwxrwx   2 nobody   nogroup    131072 Jun 13 12:19 /apps/localDns/caddy/data
      165    128 drwxrwxrwx   2 nobody   nogroup    131072 Jun 13 12:30 /apps/localDns/coredns
      168    128 -rwxrwxrwx   1 nobody   nogroup       481 Jun 13 13:52 /apps/localDns/coredns/Corefile
root@DockerHost:~# 
```

Sometimes caddy's data is unable to write due to ownership issues, so we need to use chown to ensure the user `1000` owns this folder, as the folders are from main PVE machine, we will create a new folder and give it the ownership for write access:
```sh
mkdir -p /appdata/localDns/caddy/data2
chown 1000:1000 /appdata/localDns/caddy/data2
```

If we are using `/appdata/localDns/caddy/data2` as our main 'data' folder, then you can find root certificates in '`/appdata/localDns/caddy/data2/caddy/pki/authorities/local/`' folder.

### Create the container

Now, we need to create the stack for coreDns and caddy in a single docker compose or portainer stack file:
```yml
version: "3.8"

services:
  coredns:
    image: coredns/coredns:latest
    container_name: dns_coredns
    restart: unless-stopped
    ports:
      # Binds CoreDNS cleanly on port 53 ONLY to your physical LAN IP interface
      # This prevents any binding collisions with Netbird running on your mesh interface
      - "192.168.1.100:53:53/udp"
      - "192.168.1.100:53:53/tcp"
    volumes:
      # Maps your custom regex configuration directly into the CoreDNS container runtime path in a read-only mode(ro)
      - /apps/localDns/coredns/Corefile:/Corefile:ro
    command: -conf /Corefile

  caddy:
    image: caddy:latest
    container_name: forward_proxy_caddy
    restart: unless-stopped
    ports:
      # Listens on Port 80 across all interfaces (both LAN and Netbird)
      # This allows Caddy to catch incoming traffic no matter which network path is used
      - "192.168.1.100:80:80/tcp" # http
      - "192.168.1.100:443:443/tcp" # https
    volumes:
      # Mapped to match your actual system configuration path structure
      - /apps/localDns/caddy/config/Caddyfile:/etc/caddy/Caddyfile:ro
      - /appdata/localDns/caddy/data2:/data
      # - /apps/localDns/caddy/data:/data # or /appdata/localDns/caddy/data2 if you encounter permission issue
      - /apps/localDns/caddy/config:/config
```

depending on whether we are using docker compose or portainer stack, load and run the container.

> see [[All Published Notes/Homelab/010c opening Docker in host machine#Setting up portainer on LXC\|portainer setup]] and [[All Published Notes/Homelab/010 Self hosted Task Manager - Vikunja\|installing vikunja]] for steps if unsure how to setup and run a stack.


![038 portainer local-dns.png](/img/user/All%20Published%20Notes/Homelab/Images/038%20portainer%20local-dns.png)

Now that the stack is ready, we will run the test on local LXC:
```sh
root@DockerHost:~# docker restart dns_coredns
dns_coredns
root@DockerHost:~# sleep 2
root@DockerHost:~# nslookup portainer.tglservice 192.168.1.100
Server:         192.168.1.100
Address:        192.168.1.100#53

Name:   portainer.tglservice
Address: 192.168.1.100

root@DockerHost:~# nslookup google.com 192.168.1.100
Server:         192.168.1.100
Address:        192.168.1.100#53

Non-authoritative answer:
Name:   google.com
Address: 142.251.223.238
Name:   google.com
Address: 2404:6800:4007:810::200e
```

if you get errors like `SERVFAIL` or `NXDOMAIN`, I can only wish you luck as fixing it took a long time. I have though written only the setup that does not fail on my machine.
### Define DNS setting in devices
#### Defining the DNS setting in my PC(over Lan)

Each system that want to use this DNS setting needs to change the setting to use it. 

##### clean setup
If the system has no other settings:
The DNS IP is `192.168.1.100`, so we will configure it in `/etc/resolv.conf` if no other settings are made:
```conf
nameserver 192.168.1.100
```

After editing `/etc/resolv.conf`, we might need to update the reference or restart the device.
##### complex setup
If you have other settings in your machine, like I have made `/etc/resolv.conf` as a link to `/run/systemd/resolve/stub-resolv.conf` based on `systemd-resolved` as well as I am using netbird for virtual lan, though I will be disconnecting that when using the current setup.

In which case, better to check online on how the settings are changed for your setup. I will be writing based on my situation, as this will help gain more ideas for people facing similar situations.

**My current setup**:
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ resolvectl status
Global
         Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (enp7s0)
    Current Scopes: none
         Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 3 (wlp14s0)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: fe80::e36:23ff:fe89:2d90
       DNS Servers: 192.168.1.1 fe80::e36:23ff:fe89:2d90

Link 4 (wt0)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 100.77.182.217
       DNS Servers: 100.77.182.217
        DNS Domain: netbird.cloud ~77.100.in-addr.arpa ~.
thegamelearner@thegamelearner-MS-7E12 ~ $ ls -l /etc/resolv.conf
lrwxrwxrwx 1 root root 39 Feb 24  2025 /etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
thegamelearner@thegamelearner-MS-7E12 ~ $ ls -l /run/systemd/resolve/stub-resolv.conf
-rw-r--r-- 1 systemd-resolve systemd-resolve 932 Jun 13 21:11 /run/systemd/resolve/stub-resolv.conf
thegamelearner@thegamelearner-MS-7E12 ~ $ cat /run/systemd/resolve/stub-resolv.conf
# This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search netbird.cloud
thegamelearner@thegamelearner-MS-7E12 ~ $
```

I need to first find my active internet profile:
```sh
nmcli connection show --active
```
->
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ nmcli connection show --active
NAME                   UUID                                  TYPE       DEVICE  
Auto Airtel_vidh_2470  c8f010ef-0f1f-43e0-9e48-002799f6b0cc  wifi       wlp14s0 
lo                     a0fb84fc-d29a-48d5-8a09-5f5fb425baca  loopback   lo      
wt0                    6472053c-37e8-45d7-8190-b885772e3500  wireguard  wt0     
thegamelearner@thegamelearner-MS-7E12 ~ $ hostname -I
192.168.1.6 100.77.182.217 2401:4900:1cb8:4aaf:e900:6655:41c3:83de 2401:4900:1cb8:4aaf:5958:da53:6145:129f 
thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

I need to update the profile logic to ignore the DNS pushed by the IP provider(airtel) when the domain has the keyword `tglservice` and to use the DNS that I configure for this scenario:

```sh
nmcli connection modify "Auto Airtel_vidh_2470" ipv4.dns "192.168.1.100 1.1.1.1"
nmcli connection modify "Auto Airtel_vidh_2470" ipv4.dns-search "~tglservice"
nmcli connection modify "Auto Airtel_vidh_2470" ipv4.ignore-auto-dns no
nmcli connection modify "Auto Airtel_vidh_2470" ipv6.ignore-auto-dns no
```

Re-connect the wifi:
```sh
sudo resolvectl flush-caches
nmcli connection down "Auto Airtel_vidh_2470" && nmcli connection up "Auto Airtel_vidh_2470"
```

Verify the New Order of Operations:
```sh
resolvectl status
```
Ensure within this status, the connection(`wlp14s0`) gives DNS as the new DNS setting you set.
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ resolvectl status
Global
         Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (enp7s0)
    Current Scopes: none
         Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 3 (wlp14s0)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 192.168.1.100
       DNS Servers: 192.168.1.100 1.1.1.1 192.168.1.1 fe80::e36:23ff:fe89:2d90
        DNS Domain: ~tglservice

Link 4 (wt0)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 100.77.182.217
       DNS Servers: 100.77.182.217
        DNS Domain: netbird.cloud ~77.100.in-addr.arpa ~.
thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

We now test if we can access both homelab domain based IP and normal pages from terminal:
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup portainer.tglservice
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   portainer.tglservice
Address: 192.168.1.100

thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup google.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.182.78
Name:   google.com
Address: 2404:6800:4007:83a::200e

thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

now, from your local machine hitting '[http://browser.tglservice](http://portainer.tglservice/)' should open the portainer container without any errors.

> [!Warning]
> If you also have a virtual LAN that has it's own DNS like Netbird, the test expects you to disable it.

#### Defining the DNS setting in my virtual Network(over Netbird)

> [!Warning]
> The DNS through Netbird Need to update `iptables` in the LXC on which we are deploying the setup


Open the NameServer tab(https://app.netbird.io/dns/nameservers) on left for adding this DNS. 
Click on "Add Nameserver"
- add IP as `100.77.8.142` which is the IP of the LXC in Netbird
- and port as the standard port we defined in `Corefile`
- Distribution groups are the ones who can use this DNS setting
![039 nameserver ip.png](/img/user/All%20Published%20Notes/Homelab/Images/039%20nameserver%20ip.png)

![[039 nameserver ip.png.png\|039 nameserver ip.png.png]]

Go to Domains:
- As we only want to use this domain for `tglservice`, add that
![040 nameserver domain.png](/img/user/All%20Published%20Notes/Homelab/Images/040%20nameserver%20domain.png)

Name this Server:
![041 nameserver desc.png](/img/user/All%20Published%20Notes/Homelab/Images/041%20nameserver%20desc.png)

Now, when we are trying to access the system with Netbird, the request is coming on `100.77.8.142:53` but we are only listening on `192.168.1.100`.

##### Defining the IP forwarding in my LXC

> [!Warning]
> I tried making 2 containers, one listening on local LAN and the other on netbird ip, but had problem with errors as netbird IP was not found in iptables which blocked the container from being created with error 500.

Now, we will update `iptables` to add a rule that all requests on port *53*, *80* or *443* of `100.77.8.142:53` should be forwarded to same ports of `192.168.1.100`.
We want all incoming (PREROUTING), outgoing(POSTROUTING) or requests made by same LXC(OUTPUT) to be directed to `192.168.1.100` without fail, so we also add rule for udp and tcp.

```sh
# 1. Intercept incoming traffic from external Netbird peers (Main PC, Phone, etc.)
sudo iptables -t nat -A PREROUTING -d 100.77.8.142 -p tcp -m multiport --dports 53,80,443 -j DNAT --to-destination 192.168.1.100
sudo iptables -t nat -A PREROUTING -d 100.77.8.142 -p udp -m multiport --dports 53,80,443 -j DNAT --to-destination 192.168.1.100

# 2. Intercept local requests generated directly inside this specific LXC terminal loop
sudo iptables -t nat -A OUTPUT -d 100.77.8.142 -p tcp -m multiport --dports 53,80,443 -j DNAT --to-destination 192.168.1.100
sudo iptables -t nat -A OUTPUT -d 100.77.8.142 -p udp -m multiport --dports 53,80,443 -j DNAT --to-destination 192.168.1.100

# 3. Masquerade translated traffic so the containers can find their way back through the tunnel network flawlessly
sudo iptables -t nat -A POSTROUTING -d 192.168.1.100 -p tcp -m multiport --dports 53,80,443 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -d 192.168.1.100 -p udp -m multiport --dports 53,80,443 -j MASQUERADE
```


To validate it, open a browser and type the custom url in a device which is connected to netbird and does not have any dns setting made like a phone or another device on same wifi.

Connected to **same wifi**:
- When I try to open 'http://portainer.tglservice' on my chromebook which is connected to same wifi but not connected to the netbird, there is no DNS to process it as I have not updated the `/run/systemd/resolve/stub-resolv.conf` file or updated my `nmcli connection` details.
- When I try to open 'http://portainer.tglservice' on my chromebook which is connected to same wifi and also connected to the netbird account, I can open the link and get the admin page.
Connected to Mobile Data:
- when I try when connected to mobile data on phone without connecting to netbird VPN, I cannot open any page.
- when I try when connected to mobile data on phone and also to netbird VPN, I can open the page.

### Test

#### Issues
##### Issue 01:
The tests revealed that any url that needs `https` rather than `http` will fail, but as most of my requests only need http, i can manage the others with IP addresses.
See: [[All Published Notes/Homelab/015b Trusting Caddy Certificates\|015b Trusting Caddy Certificates]] to get an idea.

##### Issue 02:
If we restart the homelab and system, and if there is an issue, sometimes the connection does not work automatically, in which case we need to flush and reconnect. 
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ 
thegamelearner@thegamelearner-MS-7E12 ~ $ sudo resolvectl flush-caches
[sudo] password for thegamelearner:           
thegamelearner@thegamelearner-MS-7E12 ~ $ 
thegamelearner@thegamelearner-MS-7E12 ~ $ 
thegamelearner@thegamelearner-MS-7E12 ~ $ nmcli connection down "Auto Airtel_vidh_2470" && nmcli connection up "Auto Airtel_vidh_2470"
Connection 'Auto Airtel_vidh_2470' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/2)
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
thegamelearner@thegamelearner-MS-7E12 ~ $ 
thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

This problem is mostly resolved if we are also connected to netbird as it is already managing the DNS on machine, but if it is not connected, we can flush the cache and the connection will be smoothly established. 

##### Issue 03:
**NetworkManager** and **systemd-resolved** handle single-label domains (like `.tglservice`) entirely differently than proper Top-Level Domains (like `tglservice.top`) so, if we are purchasing a top level domain, we will need to resolve it differently.
See: [[All Published Notes/Homelab/015b Trusting Caddy Certificates\|015b Trusting Caddy Certificates]] to get the solution.







---

[^1]:
[^2]:

