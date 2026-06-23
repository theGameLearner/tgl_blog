---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/015b-trusting-caddy-certificates/"}
---

created: 2026-06-16
updated: 2026-06-21

### Problem with https services

As caddy is running locally, all https urls that we have may not work very well given that our local machine does not trust a random website for security. Our local browsers also have it's own security features and trusting a random site is not something they do. 

So to fix this issue, there are ways to save a certificate on your machine and then make your browser to trust it, but it should only be done when you would rather not use direct IP address.

For E.g., I can open "http://192.168.1.100:10001/" which is the IP of brave browser container, but I cannot open "https://browser.tglservice/" or "http://browser.tglservice/" which is the self-created domain for the same as I get the error "This site can’t provide a secure connection" in a browser while the nslookup still works.
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup browser.tglservice 
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   browser.tglservice
Address: 192.168.1.100

thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

**Http block** in caddy file:
```sh
# 3. Isolated Dedicated Browser Container
http://browser.tglservice {
    # tls internal # creates an internal certificate for secure access
    log {
        output stdout
    }

    reverse_proxy {
        # Upstream targets
        to http://192.168.1.100:10000 http://100.77.8.142:10000
     
        # Configure reverse proxy network layers to skip self-signed SSL errors
        transport http {
            tls_insecure_skip_verify
        }
           
        lb_policy first
        fail_duration 5s
        max_fails 1
    }
}
```

Notice that the url has no https and that we are connecting to port 10000 which is the http port of the container's port 3000.

**Solution**: 
We need a certificate that the computer and browser can trust to maintain a secure access. If we use the certificate created by caddy, we will have to add the certificate to each phone or pc that wants to connect to this setup. Alternatively, if we have a domain(web url domain), we can get a publicly trusted certificate that we can use to authorize the connection and make the secure connection.

*Steps for using Caddy certificates*:
- Get the root certificate made by caddy in `/appdata/localDns/caddy/data2/caddy/pki/authorities/local/root.crt`
- copy the new certificate file to `/usr/local/share/ca-certificates/` folder in the machine which should trust this certificate.
- sudo update-ca-certificates
- On a browser, open the "Authorities" section and add the newly added certificate “Trust this CA to identify websites”.
*Cons*:
* In each device where you want to access the services, you will have to manually trust the certificates, so any new device you bring home and connect to LAN will not be able to access the services.


For creating a certification that is publically trusted by any new device, we need a public domain. I am using a `.top` domain I purchased for 3 years(will expire in 18th June 2029) from [spaceship](https://www.spaceship.com/) called 'tglservice.top'[^1]. It would be better to buy a domain which can be purchased from cloudflare or at least transferred to cloudflare due to it's popularity.

![042 price of domain.png](/img/user/All%20Published%20Notes/Homelab/Images/042%20price%20of%20domain.png)

*Steps for using publicly trusted certificates*:
- Generate Your Spaceship [API](https://www.spaceship.com/application/api-manager/) Keys
- Update caddy's yaml file
	- we can use a file in docker compose
	- we can create a custom image and use that in portainer
		- Custom image requires the image created will format requests correctly.
	- **Use acme.sh container to get certificates**
- update `caddyfile` to change the local urls with urls using the new domain.
- update `Corefile` to use local IP with the updated url with domain
- restart the setup


> [!Note]
> I got to know that this idea of getting a secure certificate for HTTPS access is called **ACME DNS-01 challenge**. 

#### How to use purchased domain for TLS certificate
##### Generate Your Spaceship API Keys
open 'https://www.spaceship.com/application/api-manager/' and you will see a page like this:
![043 spaceship api manager.png](/img/user/All%20Published%20Notes/Homelab/Images/043%20spaceship%20api%20manager.png)
Now, add a new API key, for my current need I need only 2 things from my API:
- DNS Record
	- Read
	- Write
- Domains
	- Read
	- write
All other options are currently not needed and can be turned off
![044 custom options for API secret.png](/img/user/All%20Published%20Notes/Homelab/Images/044%20custom%20options%20for%20API%20secret.png)
Once you click on Create API Key, it will create the Key but you will have to authorize this with Spaceship password.
![045 secrets for API.png](/img/user/All%20Published%20Notes/Homelab/Images/045%20secrets%20for%20API.png)

Copy and save the key as you will not see them again.
We need these keys as environment variables in the Caddy's compose file or yml file.

> [!Note]
> Test the keys and try to connect, it will save a lot of time

**Optional**: using the keys, you can test the connection on a terminal:
```sh
root@DockerHost:~# # Old approach
root@DockerHost:~# curl -X GET "https://spaceship.dev/api/v1/domains" \
  -H "Authorization: Bearer abc" \
  -H "X-API-Secret: xyz"
{"detail":"Api key or secret not provided."}root@DockerHost:~# 
root@DockerHost:~# 
root@DockerHost:~# # Another approach
root@DockerHost:~# curl -X GET "https://spaceship.dev/api/v1/domains" \
  -H "X-Api-Key: abc" \
  -H "X-Api-Secret: xyz"
{"detail":"The request is invalid.","data":[{"field":"take","details":"The take field is required."},{"field":"skip","details":"The skip field is required."}]}root@DockerHost:~# 
root@DockerHost:~#  
```

> [!Note]
> Make sure to use `X-Api-Key` and `X-Api-Secret` for authorization, as using 'Authorization' does not authorize the user.

##### Update Caddy's yml file

*The standard Caddy Docker image does not contain the specialized `spaceship` DNS binary plugin. Instead of downloading untrusted third-party images, we can configure Docker Compose to build a secure binary natively on boot using standard Docker multi-stage instructions.*[^2]

> [!Warning]
> This approach 'may' work for Docker compose users, for portainer, this will most likely fail. I do not know the right solution for this approach. Another problem is that we are using `caddy-dns/spaceship` plugin[^3] which may or may not work properly.

Go to portainer(preferrably with IP address as we are changing caddy and coreDNS - 'http://192.168.1.100:9443/')
Stop the caddy(forward_proxy_caddy) and CoreDNS(dns_coredns) container.
Stop the stack(local-dns).
Update the yml from
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
      - "80:80" # http
      - "443:443" # https
    volumes:
      # Mapped to match your actual system configuration path structure
      - /apps/localDns/caddy/config/Caddyfile:/etc/caddy/Caddyfile:ro
      - /appdata/localDns/caddy/data2:/data
      # - /apps/localDns/caddy/data:/data # or /appdata/localDns/caddy/data2 if you encounter permission issue
      - /apps/localDns/caddy/config:/config
```
to:
```yml
version: "3.9"

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
    build: # you are telling Docker Compose: "Don't download a pre-made image from the internet. Build me a custom image on my local computer instead."
      context: .
      dockerfile_inline: |
        FROM caddy:2.7-builder-alpine AS builder
        RUN xcaddy build --with github.com/caddy-dns/spaceship
        FROM caddy:2.7-alpine
        COPY --from=builder /usr/bin/caddy /usr/bin/caddy
    container_name: forward_proxy_caddy
    restart: unless-stopped
    environment:
      - SPACESHIP_API_KEY=
      - SPACESHIP_API_SECRET=
    ports:
      # Listens on Port 80 across all interfaces (both LAN and Netbird)
      # This allows Caddy to catch incoming traffic no matter which network path is used
      - "80:80" # http
      - "443:443" # https
    volumes:
      # Mapped to match your actual system configuration path structure
      - /apps/localDns/caddy/config/Caddyfile:/etc/caddy/Caddyfile:ro
      - /appdata/localDns/caddy/data2:/data
      # - /apps/localDns/caddy/data:/data # or /appdata/localDns/caddy/data2 if you encounter permission issue
      - /apps/localDns/caddy/config:/config
```

Here, we are removing `image` section to use `build` section. This tells the machine, do not use a pre-build image from the internet, rather, download a image we want and build on top of it so that the setup is exactly as we need. We also use version 2.7 even though 2.11 is out, because AI (Gemini) that I am using is sure of this code rather than the latest version.

We are also using *DNS provider module*[^3] for caddy when using spaceship.

> [!Note]
> Setup with cloudflare is easier, I bought `.top` domain which cloudflare does not support, so I have to use other ways to make it work with spaceship.

> [!Warning]
> The setup fails, because directly trying to `build` inside portainer without proper `context` fails.

Now, to have a built data, we have 2 options:
- create an image with the necessary setting and use that image in caddy file
	- Once created, the image is safe in docker files
- Create a simple file at `/apps/localDns/Dockerfile` which we use as `dockerfile` and `context`
	- We can always correct the file and re-deploy the stack(container) without extra changes at container level
###### using a file as context
I obviously, chose to create a file as I needed the backup for future:
```sh
root@DockerHost:~# 
root@DockerHost:~# ls -al /apps/localDns/
total 512
drwxrwxrwx 4 nobody nogroup 131072 Jun 13 11:57 .
drwxrwxrwx 3 nobody nogroup 131072 Jun 13 11:56 ..
drwxrwxrwx 4 nobody nogroup 131072 Jun 13 12:19 caddy
drwxrwxrwx 2 nobody nogroup 131072 Jun 13 12:30 coredns
root@DockerHost:~# fresh /apps/localDns/Dockerfile

A new version of fresh is available: 0.3.5 -> 0.4.1
Download from: https://github.com/sinelaw/fresh/releases/tag/v0.4.1

root@DockerHost:~# cat /apps/localDns/Dockerfile

FROM caddy:2.7-builder-alpine AS builder

RUN xcaddy build --with github.com/caddy-dns/spaceship

FROM caddy:2.7-alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy

root@DockerHost:~# 
```

then in the yml file, update the `build` section:
```yml
    build: # you are telling Docker Compose: "Don't download a pre-made image from the internet. Build me a custom image on my local computer instead."
      context: /apps/localDns
      dockerfile: Dockerfile
    container_name: forward_proxy_caddy
```

Unfortunately for me, this alone was not enough and updating the yaml file failed again.
*I assume the reason of failure is that this needs some other changes or permissions, but the concept is not clear enough to fix*. **Another possiblity is failure of `caddy-dns/spaceship` plugin**.

Deleting this file, moving to next approach.

###### using a custom image for caddy source

I will create a file to be used as docker image for a caddy that can talk to spaceship API:
```sh
root@DockerHost:~# fresh /apps/localDns/Dockerfile.caddy
root@DockerHost:~# cat /apps/localDns/Dockerfile.caddy
FROM caddy:builder-alpine AS builder
RUN xcaddy build --with github.com/caddy-dns/spaceship
FROM caddy:alpine
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
root@DockerHost:~# 
```

Now, we run this file and create an image using docker terminal commands:
```sh
root@DockerHost:~# docker build -t custom-caddy:spaceship -f /apps/localDns/Dockerfile.caddy /apps/localDns
```
This tells docker to build a custom image with name `custom-caddy:spaceship` which will use `/apps/localDns` as the context and `/apps/localDns/Dockerfile.caddy` as the source for what to do in the file.
This creates a new image in your portainer which can be used as the base for the new caddy file.
![046 Custom image for caddy.png](/img/user/All%20Published%20Notes/Homelab/Images/046%20Custom%20image%20for%20caddy.png)

Using this image, we create the new yml file:
```yml
version: "4.0"

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
    image: custom-caddy:spaceship # custom created image
    container_name: forward_proxy_caddy
    restart: unless-stopped
    environment:
      - SPACESHIP_API_KEY=
      - SPACESHIP_API_SECRET=
    ports:
      # Listens on Port 80 across all interfaces (both LAN and Netbird)
      # This allows Caddy to catch incoming traffic no matter which network path is used
      - "80:80" # http
      - "443:443" # https
    volumes:
      # Mapped to match your actual system configuration path structure
      - /apps/localDns/caddy/config/Caddyfile:/etc/caddy/Caddyfile:ro
      - /appdata/localDns/caddy/data2:/data
      # - /apps/localDns/caddy/data:/data # or /appdata/localDns/caddy/data2 if you encounter permission issue
      - /apps/localDns/caddy/config:/config
```

After failure, I tried to find the problem, and one possibility is that `caddy-dns/spaceship` formats the requests in wrong format.
We will now try to skip this plugin and get real "Let's Encrypt" certificates directly via Spaceship by using a lightweight utility script container inside the Portainer Stack called **acme.sh**.

###### Use acme.sh container to get certificates

We will use `acme.sh` image which sets the JSON as needed to let's encrypt standard and gets a publicly trusted certificate for 'tglservice.top' using the spaceship's API keys.
Caddy no longer needs to make API calls or create local certificates. We will switch to `caddy:2.7-alpine` instead of latest to avoid any new changes introduced in latest version.
The certificate will be placed in `/appdata/localDns/certs` folder and caddy will use the same for certificates.

```yml
version: "4.1"

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

  acme_sh:
    image: neilpang/acme.sh:latest
    container_name: cert_updater_acme_caddy 
    restart: unless-stopped
    environment:
      # Exact case-sensitive environment flags required by acme.sh
      - SPACESHIP_API_KEY=
      - SPACESHIP_API_SECRET=
    volumes:
      - /appdata/localDns/certs:/acme.sh
    # Added --keylength ec-256 to ensure it builds an ECC cert matching the Caddy paths
    entrypoint: /bin/sh -c "acme.sh --upgrade --auto-upgrade && acme.sh --issue --dns dns_spaceship -d tglservice.top -d '*.tglservice.top' --keylength ec-256 --server letsencrypt; crond -f"

  caddy:
    image: caddy:2.7-alpine
    container_name: forward_proxy_caddy
    restart: unless-stopped
    ports:
      # Listens on Port 80 across all interfaces (both LAN and Netbird)
      # This allows Caddy to catch incoming traffic no matter which network path is used
      - "80:80" # http
      - "443:443" # https
    volumes:
      # Mapped to match your actual system configuration path structure
      - /apps/localDns/caddy/config/Caddyfile:/etc/caddy/Caddyfile:ro
      - /appdata/localDns/caddy/data2:/data
      # - /apps/localDns/caddy/data:/data # or /appdata/localDns/caddy/data2 if you encounter permission issue
      - /apps/localDns/caddy/config:/config
      - /appdata/localDns/certs:/certs:ro
```

> [!Note]
> we might be able to remove `/appdata/localDns/caddy/data2:/data` but I am keeping it for now.


##### update `caddyfile`
update your `/apps/localDns/caddy/config/Caddyfile` to show the URLs now use the new domain (`*.tglservice.top`) along with the path for the certificates(`/appdata/localDns/certs` or `/certs` as defined inside caddy yml file):
```sh
root@DockerHost:~# fresh /apps/localDns/caddy/config/Caddyfile
root@DockerHost:~# cat -n /apps/localDns/caddy/config/Caddyfile
     1  # =====================================================================
     2  # GLOBAL OPTIONS BLOCK
     3  # =====================================================================
     4  {
     5      # Stops Caddy from trying to request public Let's Encrypt certs for private IPs
     6      local_certs
     7  }
     8
     9  # Settings placed here apply to the entire Caddy server globally.
    10  # {
    11      # 0. Disable automatic HTTPS for local domains (stops Caddy from trying 
    12      # to obtain public Let's Encrypt certificates for internal URLs)
    13      # auto_https off
    14      # 
    15      # --- ADVANCED GLOBAL OPTION EXAMPLES (Commented for future reference) ---
    16      #
    17      # 1. Change the admin interface bind address or disable it completely:
    18      # admin off
    19      # admin 127.0.0.1:2019
    20      #
    21      # 2. Set default HTTP/HTTPS ports if you are customizing host binds:
    22      # http_port 80
    23      # https_port 443
    24      #
    25      # 3. Configure global logging formats or send files to specific paths:
    26      # log {
    27      #     output file /var/log/caddy/access.log {
    28      #         roll_size 10mb
    29      #         roll_keep 5
    30      #     }
    31      # }
    32      #
    33      # 4. Adjust global timeout parameters for connections:
    34      # servers {
    35      #     timeouts {
    36      #         read_body   10s
    37      #         read_header 10s
    38      #         write       10s
    39      #         idle        2m
    40      #     }
    41      # }
    42  # }
    43
    44  # =====================================================================
    45  # DEFINE YOUR SNIPPET (ONCE AT THE VERY TOP)
    46  # defines the certification path local to caddy container
    47  # defines the common setup needed for a url
    48  # {args.0} = Main LAN IP & Port (e.g., 192.168.1.100:9443)
    49  # {args.1} = Netbird Mesh IP & Port (e.g., 100.77.8.142:9443)
    50  # ===================================================================== 
    51
    52  (secure_proxy_https) {
    53      # Load your Let's Encrypt Wildcard Certificates
    54      tls /certs/tglservice.top_ecc/fullchain.cer /certs/tglservice.top_ecc/tglservice.top.key
    55      
    56      # Standardize access logs to stdout - Send access logs directly to stdout (visible in Docker/Portainer logs)
    57      log {                           
    58          output stdout               
    59      }
    60
    61      # Route traffic using your priority failover engine
    62      reverse_proxy {
    63          to {args[0]} {args[1]}
    64          
    65          transport http {    
    66              tls_insecure_skip_verify
    67              # We want to stop checking the certificate when 
    68              # we are connecting to a service on an IP address
    69              # this is between caddy and the destination IP
    70          }                           
    71                                      
    72          lb_policy first             
    73          fail_duration 5s            
    74          max_fails 1                 
    75      }
    76  }
    77
    78  (secure_proxy_http) {
    79      # Load your Let's Encrypt Wildcard Certificates
    80      tls /certs/tglservice.top_ecc/fullchain.cer /certs/tglservice.top_ecc/tglservice.top.key
    81      
    82      # Standardize access logs to stdout - Send access logs directly to stdout (visible in Docker/Portainer logs)
    83      log {                           
    84          output stdout               
    85      }
    86
    87      # Route traffic using your priority failover engine
    88      reverse_proxy {
    89          to {args[0]} {args[1]}
    90                                      
    91          lb_policy first             
    92          fail_duration 5s            
    93          max_fails 1                 
    94      }
    95  }
    96
    97
    98  # =====================================================================
    99  # REVERSE PROXY & HOST DEFINITIONS
   100  # Note: 
   101  #    - Since Vikunja uses standard HTTP upstream, we append "http://" 
   102  #      to the args to tell Caddy not to use HTTPS on the back-end connection.
   103  #    - Browser uses secure connection so we use "https://"
   104  # =====================================================================
   105
   106  # 1. Vikunja Task Management Service  
   107  vikunja.tglservice.top {
   108      import secure_proxy_http http://192.168.1.100:3456 http://100.77.8.142:3456
   109  }
   110           
   111  # 2. Portainer Management Dashboard
   112  portainer.tglservice.top {         
   113      import secure_proxy_https https://192.168.1.100:9443 https://100.77.8.142:9443
   114  }        
   115           
   116  # 3. Isolated Dedicated Browser Container
   117  browser.tglservice.top {                 
   118      import secure_proxy_https https://192.168.1.100:10001 https://100.77.8.142:10001
   119  }        
   120           
   121  # ===================================================================== 
   122  # UTILITY AND FALLBACK BLOCKS
   123  # =====================================================================
   124  # 4. Global Health Checker
   125  health.tglservice.top {   
   126      tls /certs/tglservice.top_ecc/fullchain.cer /certs/tglservice.top_ecc/tglservice.top.key
   127      log { 
   128          output stdout 
   129      } 
   130      respond "OK" 200      
   131  }        
   132
   133  # 5. Catch-All Default 404 Error Block
   134  *.tglservice.top, tglservice.top {
   135      tls /certs/tglservice.top_ecc/fullchain.cer /certs/tglservice.top_ecc/tglservice.top.key
   136      log {
   137          output stdout
   138      }
   139      respond "Service Not Found" 404 
   140  #    {
   141  #        close    
   142  #    }        
   143  }
   144
root@DockerHost:~# 
```

As you can see, we removed `http://` from all URL and added `.top` at end to match the domain we purchased, also at the top `auto_https off` is commented now. We also added the security keys in `acme_dns` section.

Verify with docker's status of 'forward_proxy_caddy' to ensure there is no issue in caddyfile :
```sh
root@DockerHost:~# docker restart forward_proxy_caddy
forward_proxy_caddy
root@DockerHost:~# docker ps | grep caddy
c7a594431ffa   neilpang/acme.sh:latest            "/bin/sh -c 'acme.sh…"   23 minutes ago   Up 23 minutes                                                                                                        cert_updater_acme_caddy
93ce91ee0800   caddy:2.7-alpine                   "caddy run --config …"   23 minutes ago   Up 2 seconds    0.0.0.0:80->80/tcp, [::]:80->80/tcp, 0.0.0.0:443->443/tcp, [::]:443->443/tcp, 443/udp, 2019/tcp      forward_proxy_caddy
root@DockerHost:~# docker logs forward_proxy_caddy
```
In case of error, we will get something other than `Up 2 seconds` in `forward_proxy_caddy`'s log.
we can read the errors by `docker logs forward_proxy_caddy`.

##### Update the `corefile`
Similarly, let's update your `/apps/localDns/coredns/Corefile` config:
```sh
root@DockerHost:~# cat /apps/localDns/coredns/Corefile
.:53 {
    log
    errors
    
    # Use hosts file for exact tglservice names (no regex needed)
    hosts {
        192.168.1.100 portainer.tglservice.top
        192.168.1.100 vikunja.tglservice.top
        192.168.1.100 browser.tglservice.top
        192.168.1.100 health.tglservice.top
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

Now we can restart the containers in portainer.

##### Testing in terminal
In the LXC where we added the containers, use the LXC's IP as a DNS router IP to lookup the service:
```sh
root@DockerHost:~# nslookup portainer.tglservice.top
Server:         127.0.0.53
Address:        127.0.0.53#53

** server can't find portainer.tglservice.top: NXDOMAIN

root@DockerHost:~# nslookup portainer.tglservice.top 192.168.1.100
Server:         192.168.1.100
Address:        192.168.1.100#53

Name:   portainer.tglservice.top
Address: 192.168.1.100

root@DockerHost:~# 
```

This means, our LXC can find the service with the specified DNS, but not without it.

Run the same test in the main PC
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup portainer.tglservice.top
Server:         127.0.0.53
Address:        127.0.0.53#53

** server can't find portainer.tglservice.top: NXDOMAIN

thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup portainer.tglservice.top 192.168.1.100
Server:         192.168.1.100
Address:        192.168.1.100#53

Name:   portainer.tglservice.top
Address: 192.168.1.100

thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

Remember to flush old connections ([[All Published Notes/Homelab/015 Managing custom DNS in my homelab#**Issue 02**\|flushing domains local data]]) as without it we can get failures.

#### How to use the new domain

Next, we need to ensure the DNS we are using is able to re-direct the DNS query to the desired IP when the domain is (`*.tglservice.top`). 
We have done this in [[All Published Notes/Homelab/015 Managing custom DNS in my homelab#Defining the DNS setting in my PC(over Lan)\|Defining DNS Settings]] and will do something similar now, but changing `*.tglservice` to `*.tglservice.top` will make our domain a real TLD(top level domain) which is fetched differently than a single label domain like `*.tglservice`.

Before we cover how the setup changes, let us first see what happens if we try to fetch the domain like before:

```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ nmcli connection modify "Auto Airtel_vidh_2470" ipv4.dns-search "~tglservice.top"
thegamelearner@thegamelearner-MS-7E12 ~ $ sudo resolvectl flush-caches
[sudo] password for thegamelearner:           
thegamelearner@thegamelearner-MS-7E12 ~ $ nmcli connection down "Auto Airtel_vidh_2470" && nmcli connection up "Auto Airtel_vidh_2470"
Connection 'Auto Airtel_vidh_2470' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/2)
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup portainer.tglservice.top
Server:         127.0.0.53
Address:        127.0.0.53#53

** server can not find portainer.tglservice.top: NXDOMAIN

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
Current DNS Server: fe80::e36:23ef:fd89:2e90
       DNS Servers: 192.168.1.100 1.1.1.1 192.168.1.1 fe80::e36:23ef:fd89:2e90
        DNS Domain: ~tglservice.top

Link 4 (wt0)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 100.77.182.217
       DNS Servers: 100.77.182.217
        DNS Domain: ~tglservice.top netbird.cloud ~77.100.in-addr.arpa ~.
thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

This will most likely fail as seen above, but you can see in `resolvectl status` output that the machine is using `fe80::e36:23ef:fd89:2e90` as the DNS server. When our domain was a single label, setting the search label worked as no actual DNS could solve the label, now that we have a proper domain, the actual IPv6 DNS will result in `NXDOMAIN` and as a result, the system never goes to our custom DNS for a solution.

If we go and remove the default IPv6 DNS which is pushed from the router, `thegamelearner@thegamelearner-MS-7E12 ~ $ nmcli connection modify "Auto Airtel_vidh_2470" ipv6.ignore-auto-dns yes` we will get `SERVFAIL`:
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ nmcli connection modify "Auto Airtel_vidh_2470" ipv6.ignore-auto-dns yes
thegamelearner@thegamelearner-MS-7E12 ~ $ sudo resolvectl flush-caches
[sudo] password for thegamelearner:           
thegamelearner@thegamelearner-MS-7E12 ~ $ nmcli connection down "Auto Airtel_vidh_2470" && nmcli connection up "Auto Airtel_vidh_2470"
Connection 'Auto Airtel_vidh_2470' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
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
       DNS Servers: 192.168.1.100 1.1.1.1 192.168.1.1
        DNS Domain: ~tglservice.top

Link 4 (wt0)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 100.77.182.217
       DNS Servers: 100.77.182.217
        DNS Domain: ~tglservice.top netbird.cloud ~77.100.in-addr.arpa ~.
thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup portainer.tglservice.top 192.168.1.100
Server:         192.168.1.100
Address:        192.168.1.100#53

Name:   portainer.tglservice.top
Address: 192.168.1.100

thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup portainer.tglservice.top
;; Got SERVFAIL reply from 127.0.0.53
Server:         127.0.0.53
Address:        127.0.0.53#53

** server can't find portainer.tglservice.top: SERVFAIL

thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

The problem might be that Netbird(wt0) and Wifi(wlp14s0) are both trying to resolve the same domain (`~tglservice.top`), so I will down the netbird and try again.
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ netbird down
Disconnected
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
Current DNS Server: 1.1.1.1
       DNS Servers: 192.168.1.100 1.1.1.1 192.168.1.1
        DNS Domain: ~tglservice.top
thegamelearner@thegamelearner-MS-7E12 ~ $ sudo resolvectl flush-caches
thegamelearner@thegamelearner-MS-7E12 ~ $ nmcli connection down "Auto Airtel_vidh_2470" && nmcli connection up "Auto Airtel_vidh_2470"
Connection 'Auto Airtel_vidh_2470' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/6)
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/7)
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
       DNS Servers: 192.168.1.100 1.1.1.1 192.168.1.1
        DNS Domain: ~tglservice.top
thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup browser.tglservice.top
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   browser.tglservice.top
Address: 192.168.1.100

thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup https://browser.tglservice.top
Server:         127.0.0.53
Address:        127.0.0.53#53

** server can not find https://browser.tglservice.top: NXDOMAIN

thegamelearner@thegamelearner-MS-7E12 ~ $ nslookup http://browser.tglservice.top
Server:         127.0.0.53
Address:        127.0.0.53#53

** server can not find http://browser.tglservice.top: NXDOMAIN

thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

Now, we can try to open the pages in a browser. This should solve the access issue.

#### How to add a new domain
To add a new domain, open the corefile(`/apps/localDns/coredns/Corefile`) and caddyfile(`/apps/localDns/caddy/config/Caddyfile`) and add the IP and domain name of the service.

### Test

#### Issues
##### Issue 01:
Everytime a new domain needs to be added we need to update 2 files, given that corefile always points to same IP, this file should be simplified, but in my current approach, i could not do it.

##### Issue 02:
We have disabled IPv6 DNS from the machine, but it is the standard DNS for most machines, we can create another IPv6 DNS in the containers, but that is beyond what we are doing here.
We can use the domain itself to direct the traffic to the desired IP without changing any DNS setting. See: [[All Published Notes/Homelab/015c Making Domain accessible publically\|015c Making Domain accessible publically]]
















---

[^1]: [spaceship-domains](https://www.spaceship.com/application/domain-portfolio/)
[^2]: Answer from Gemini, please check for trust as it is un-trusted.
[^3]: https://github.com/caddy-dns/spaceship
[^4]: 

