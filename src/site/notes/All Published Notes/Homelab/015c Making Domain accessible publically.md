---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/015c-making-domain-accessible-publically/"}
---

created: 2026-06-23
updated: 2026-06-23

A senior in my company said he can connect to his services even when he is not in same WiFi, without any changes on the machine accessing the site. Meaning I can type his service domain and I will be accessing his services on his Homelab while I have not done anything on my machine.

I am jealous. really jealous. Here he is able to access things on his homelab without worry, and I am setting up DNS records in my machine for accessing the same sites. I want my services to do the same.

Desired result: 
- I can use netbird to connect my devices to my services from anywhere.
- Anyone connected to my home WiFi can connect to my services.

After seaching and using google(gemini), I found that I can access this by adding a 'A' record to my domain which will give me my homelab's IP address when I try to connect to it.

> [!Note]
> A concern is that if I am not connected to netbird or my wifi, I may get an IP which will fail. But this needs to be tested.

### Changes using Homelab's actual IP address on LAN

Log into spaceship and open [Domain Manager](https://www.spaceship.com/application/domain-list-application/)
![047 Spaceship Domain manager.png](/img/user/All%20Published%20Notes/Homelab/Images/047%20Spaceship%20Domain%20manager.png)
Open DNS
![048 Spaceship Domain manager Settings.png](/img/user/All%20Published%20Notes/Homelab/Images/048%20Spaceship%20Domain%20manager%20Settings.png)

open advanced DNS
![049 Spaceship Domain manager Advance Settings.png](/img/user/All%20Published%20Notes/Homelab/Images/049%20Spaceship%20Domain%20manager%20Advance%20Settings.png)

![050 Spaceship DNS records.png](/img/user/All%20Published%20Notes/Homelab/Images/050%20Spaceship%20DNS%20records.png)

Create a new record of 'A' type, set the IP to the IP of Caddy and time to live as low value.
![051 Spaceship A record.png](/img/user/All%20Published%20Notes/Homelab/Images/051%20Spaceship%20A%20record.png)

Test:
Connecting to portainer.tglservice.top on multiple devices:

| Device name    | **Connected to same WiFi** | Connected to same Netbird account | Result  |
| -------------- | -------------------------- | --------------------------------- | ------- |
| Chromebook     | Yes                        | Yes                               | Success |
| One Plus Phone | Yes                        | No                                | Success |
| real me        | No                         | No                                | Fail    |
| Samsung        | No                         | Yes                               | Fail    |

### Changes using Homelab's Netbird IP address
Let's find the Netbird IP address (`100.*.*.*`) of the LXC which is running Docker:
```sh
root@pve:~# pct enter 401
root@DockerHost:~# hostname -I
192.168.1.100 100.77.8.142 172.17.0.1 172.18.0.1 172.19.0.1 172.20.0.1 
root@DockerHost:~# # Netbird IP address is 100.77.8.142
```

We will do the same steps as above but change the IP address from `192.168.1.100` to `100.77.8.142`.

Now we run the same tests again:

| Device name    | Connected to same WiFi | **Connected to same Netbird account** | Result  |
| -------------- | ---------------------- | ------------------------------------- | ------- |
| Chromebook     | Yes                    | Yes                                   | Success |
| One Plus Phone | Yes                    | No                                    | Fail    |
| real me        | No                     | No                                    | Fail    |
| Samsung        | No                     | Yes                                   | Success |

Meaning, we can now choose to either use Netbird to connect from anywhere, or we can connect to the services using local wifi.

Given that I am lazy to change frequently, and nearly all devices have a netbird setup already done, I will go with using Netbird IP.

#### Advanced Change
I do not feel content always using my Netbird IP, this works, but what if I have a guest at home and I want to provide them my music stream link to play songs for party? I cannot share my personal phone and we may not always want to access one computer.

I decided to change my URLs to add a subdomain called `*.nb.tglservice.top` which I will use when I am using netbird from outside my home.
To achieve this
- I will update my yml(portainer stack) to get certificate for the new subdomain as well
- In caddyfile(`/apps/localDns/caddy/config/Caddyfile`) I will add the new url with old URL
- No need to edit CoreDNS as coreDNS is a DNS for the LAN, as we are re-directing traffic from Spaceship, CoreDNS is no longer used.

![052 Stack certificate for new subdomain.png](/img/user/All%20Published%20Notes/Homelab/Images/052%20Stack%20certificate%20for%20new%20subdomain.png)

![053 Caddyfile update for new subdomain.png](/img/user/All%20Published%20Notes/Homelab/Images/053%20Caddyfile%20update%20for%20new%20subdomain.png)

### Effect of using Domain based routing
Using the actual public domain comes with a few ease of use scenarios:
- I can remove the DNS settings in netbird
- I can remove the DNS settings from my main PC as well

### Issue
I can connect from anywhere, but I am dependent on Netbird when outside my wifi. I am still not totally satisfied, but if I do not want a online server or a static IP with opened port, this seems to be the closest I can get to the desired output. I am happy with this and will not continue on reverse proxy on this topic.
















---

[^1]:
[^2]:

