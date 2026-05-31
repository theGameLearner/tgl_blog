---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/013-create-a-browser-for-search/"}
---

created: 2026-05-23
updated: 2026-05-23

To avoid tracking or leaving traces in my browser, I want to use a disposable browser so that all my history is deleted and I can use it without cookies or tracking.

As this will be another docker application which will run a web browser inside my browser, I will be using docker with portainer stack as:
```yml
version: "2"
services:
  disposable-browser:
    image: lscr.io/linuxserver/brave:latest # other options lscr.io/linuxserver/firefox:latest and lscr.io/linuxserver/tor-browser:latest 
    container_name: risky_browser
    security_opt:
      - seccomp:unconfined # Often needed for Chromium's internal sandbox inside an LXC
    environment:
      - PID=2300 # can also use PUID (dockeruser user)
      - GID=990 # can also use PGID (docker group)
      - TZ=Asia/Kolkata
      - CHROME_CLI=https://start.duckduckgo.com --incognito --no-first-run # Force incognito and privacy search
    ports:
      - 10000:3000 # HTTP Web UI Access, container will use port 3000, we will use 10000 as the port number in LXC
      - 10001:3001 # HTTPS Web UI Access
    shm_size: "2gb" # Crucial: Keeps Chromium from crashing on heavy pages
    restart: unless-stopped
```

Assuming my LXC has IP `100.77.8.142` (Netbird), I can simply access this using the URL `http://100.77.8.142:10000` or `https://100.77.8.142:10001/`.

To test it
- open portainer webpage in browser: `http://100.77.8.142:9443/`
- Select the environment in which we want to create the container: `local`
- Click on stack
- on top right of the stack list : 'Add Stack'
- Name the stack as `risky_browser` 
- Ensure Build method is "Web editor"
- paste the yaml file with any changes that you might have made
- disable 'Access Control' as only I use portainer to manage it
- Click on 'Deploy the stack'
- Open your browser to `https://100.77.8.142:10001/` and use advanced option to bypass SSL certificate issue.






---

[^1]:
[^2]:

