---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/operating-system/linux-mint/vpn/using-cloudflare-s-warp-on-linux-mint/"}
---

created: 2025-11-23
updated: 2025-11-23


## CloudFlare
Cloudflare provides warp which is free to use. I want to use it for accessing sites that are too slow otherwise.
find the packages here: https://pkg.cloudflareclient.com/
### Installation
###### add the cloudflare GPG key
Add the GPG key for singing:
```sh
# Add cloudflare gpg key
curl -fsSL https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg
```
or you can use a simpler code:
```sh

thegamelearner@thegamelearner-MS-7E12 ~/Documents/GithubNotes/TglBlog $ curl https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1680  100  1680    0     0   3338      0 --:--:-- --:--:-- --:--:--  3333
thegamelearner@thegamelearner-MS-7E12 ~/Documents/GithubNotes/TglBlog $ 
```
###### Adding the cloudflare's repo to apt repositories
Add the repository to your `/etc/apt/sources.list.d` directory as a list, but this also 'fails' on Linux Mint:
```sh
# Add this repo to your apt repositories (fails)
echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list
```
e.g.,
```sh
# This will 'fail'
thegamelearner@thegamelearner-MS-7E12 ~/Documents/GithubNotes/TglBlog $ echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-warp.list
deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ xia main
thegamelearner@thegamelearner-MS-7E12 ~/Documents/GithubNotes/TglBlog $ 
```

The previous code returned `xia` for `$(lsb_release -cs)`, but the names given by Linux Mint will vary from the Ubuntu names that cloudfare configured.
in my case, it is 'jammy' for Linux Mint 21(Ubuntu 22.04), so I can use:
```sh
# Ensure to find the correct name before using 'jammy'
thegamelearner@thegamelearner-MS-7E12 ~/Documents/GithubNotes/TglBlog $ echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ jammy main" | sudo tee /etc/apt/sources.list.d/cloudflare-warp.list
deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ jammy main
thegamelearner@thegamelearner-MS-7E12 ~/Documents/GithubNotes/TglBlog $ 
```

###### update all repositories
use `sudo apt-get update` to update all repositories, this way you can see if you have an error or cloudflare is successfully fetched even without installing.
```sh
E: The repository 'https://pkg.cloudflareclient.com xia Release' does not have a Release file.
```
In case this fails, use `sudo rm /etc/apt/sources.list.d/cloudflare-warp.list` to remove the list added and return to previous step([[#Adding the cloudflare's repo to apt repositories]]).

###### Install cloudflare
Now, we have cloudflare available as a known package, so it can be installed as:
```sh
sudo apt-get install cloudflare-warp
```

### Using Warp
see: https://developers.cloudflare.com/warp-client/get-started/linux/
#### Using first time (Register)
To connect for the very first time:

1. Register the client `warp-cli registration new`.
2. Connect `warp-cli connect`.
3. Run `curl https://www.cloudflare.com/cdn-cgi/trace/` and verify that `warp=on`.
```sh

thegamelearner@thegamelearner-MS-7E12 ~/Documents/Cloudflare $ warp-cli registration new
NOTICE:

Cloudflare only collects limited DNS query and traffic data (excluding payload)
that is sent to our network when you have the app enabled on your device. We
will not sell, rent, share, or otherwise disclose your personal information to
anyone, except as otherwise described in this Policy, without first providing
you with notice and the opportunity to consent. All information is handled in
accordance with our Privacy Policy.

More information is available at:
- https://www.cloudflare.com/application/terms/
- https://www.cloudflare.com/application/privacypolicy/

Accept Terms of Service and Privacy Policy? [y/N] y

Success
thegamelearner@thegamelearner-MS-7E12 ~/Documents/Cloudflare $ warp-cli connect
Success
thegamelearner@thegamelearner-MS-7E12 ~/Documents/Cloudflare $ curl https://www.cloudflare.com/cdn-cgi/trace/
fl=454f9
h=www.cloudflare.com
ip=2a09:bac5:3b48:11cd::1c6:9
ts=1763880526.371
visit_scheme=https
uag=curl/8.5.0
colo=MAA
sliver=none
http=http/2
loc=IN
tls=TLSv1.3
sni=plaintext
warp=on
gateway=off
rbi=off
kex=X25519
thegamelearner@thegamelearner-MS-7E12 ~/Documents/Cloudflare $ 
```

#### To Connect
use the command to connect:

```sh
thegamelearner@thegamelearner-MS-7E12 ~/Documents/Cloudflare $ warp-cli connect
Success
thegamelearner@thegamelearner-MS-7E12 ~/Documents/Cloudflare $ # Setting the connection mode
thegamelearner@thegamelearner-MS-7E12 ~/Documents/Cloudflare $ warp-cli mode warp+doh
Success
thegamelearner@thegamelearner-MS-7E12 ~/Documents/Cloudflare $
```

#### To Disconnect
use the command to disconnect:


```sh
thegamelearner@thegamelearner-MS-7E12 ~/Documents/Cloudflare $ warp-cli disconnect
Success
thegamelearner@thegamelearner-MS-7E12 ~/Documents/Cloudflare $ 
```




---

[^1]:
[^2]:

