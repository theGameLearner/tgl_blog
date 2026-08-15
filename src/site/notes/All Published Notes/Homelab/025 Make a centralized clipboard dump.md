---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/025-make-a-centralized-clipboard-dump/"}
---

created: 2026-08-08
updated: 2026-08-08

### Idea
Sometimes, we do not need a notes app, we are working on something and need to quickly make a note, maybe when talking to a friend, listening to a new song and want to save the name for future re-visit, shopping list when leaving the house, etc.

I do not want to use my Vikunja tasks for these, so making a second service for quick notes which are one and done. Bonus as the files are saved, so I can copy files between my machines in my homelab.

### Options
To achieve my idea, I liked the following options:
- [UseMemos](https://demo.usememos.com/explore) - login once after opening the link
- [pastefy](https://pastefy.app/) - 
- [ByteStash](https://github.com/jordan-dalby/ByteStash) 
	- https://bytestash-demo.pikapod.net/public/snippets 
- [Microbin](https://microbin.eu/) 
- [Gokapi](https://github.com/Forceu/gokapi) - Create a copy, get a link, use the link to access.
- [ClipCascade](https://github.com/Sathvik-Rao/ClipCascade) 

As It was important that there is no username and password so my wife can also use it, I went with [[#Gokapi]] and then [[#Microbin]] for now.

### Gokapi

Let's create a Gokapi container in Portainer:
```yml
services:
  gokapi:
    image: f0rc3/gokapi:latest
    container_name: gokapi
    ports:
      - "10006:53842"
    volumes:
      - /apps/pastebin/data:/app/data:rw
      - /apps/pastebin/config:/app/config:rw
    environment:
      - TZ=Asia/Kolkata
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "nc -z 127.0.0.1 53842 || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 90s
```


This will start the container, but Gokapi needs to be setup in a specific path before it can be used.
- Open `http://<ip_address>:10006/setup` and setup the details.
- Webserver 2/2
	- public facing url: `http://<ip_address>:10006/`
		- If you have a URL defined, use that `pastebin.nb.tglservice.top`
	- redirection url for the index: `http://<ip_address>:10006/login`
		- `pastebin.nb.tglservice.top/login`
- Authentication 
	- Username/Password: The easiest way to authenticate is username and password for making it public, you can use something else if you want to use it more securely.
- Encryption
	- I am using "0-No Encryption" as I do not want any encryption for my own home.

After "Finish", Open `http://<ip_address>:10006/admin` and log in.

> [!Warning]
> Because we are using redirect, the IP addresses may not work well when working with custom URL names.

I deleted the container, stack and all files after this as making it work on my custom URL is a necessary tool for me.
### Microbin
Same as Gopaki, craete the YML with data in same folder and using the same port:
```yml
version: '3.8'

services:
  microbin:
    image: danielszabo99/microbin:latest
    container_name: microbin
    restart: unless-stopped
    ports:
      - "10006:8080" # Format: HOST_PORT:CONTAINER_PORT
    environment:
      # Enable automatic code/syntax highlighting
      - MICROBIN_HIGHLIGHTSYNTAX=true
      
      # Allow pastes to be updated/edited later
      - MICROBIN_EDITABLE=true
      
      # Set maximum file/screenshot upload size in MB (default is 256)
      - MICROBIN_MAX_FILE_SIZE_MB=1024
      
      # Allows viewing raw text uploads (e.g. your_server:8080/raw/pasta-name)
      - MICROBIN_ENABLE_RAW=true
      
      # Show pasta listing dashboard (/pastalist)
      - MICROBIN_DISABLE_LCACHE=false
      
      # Enable public pasta creation (no login required)
      - MICROBIN_NO_DISPASS=true
      
      # Optional: Set your local timezone
      - TZ=Asia/Kolkata
    volumes:
      # Maps your host folder /apps/pastebin to Microbin's data directory
      - /apps/pastebin:/app/microbin_data:rw
```

I do not need to configure anything on MicroBin as there is no need of user authentication due to `MICROBIN_NO_DISPASS=true`

#### Add a URL to the service

Update the caddyfile and coredns configurations so that we can access this with a url
- /apps/localDns/caddy/config/Caddyfile 
- /apps/localDns/coredns/Corefile









---

[^1]: 
[^2]: 

