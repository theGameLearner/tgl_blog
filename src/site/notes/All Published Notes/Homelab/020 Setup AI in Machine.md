---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/020-setup-ai-in-machine/"}
---

created: 2026-07-12
updated: 2026-07-15

### Idea
The Idea is to make a local AI to use my homelab for coding.

The Idea:
- My Homelab has "AMD ryzen 5 5600GT" as the CPU
- I have about 12 GB of free RAM in the machine
- I also have 60 GB of space in the machine
- I will install an AI model on this machine so I can use that for coding in my Jetbrains Rider IDE.

#### Tools
The Models:
- I will use 'qwen2.5-coder:7b' or 'qwen2.5-coder:3b' as the main model inside ollama
- I will add 'aider-chat' or 'aider' in the main machine where I will use the Jetbrains Rider


### Ollama in LXC
Understand your machine, space and hardware available for use, I am not covering it in detail, but, you want to know the following:
- The RAM available to use for AI
	- I have about 12 GB
		- I will use about 8 GB if I use 'qwen2.5-coder:3b'
		- I will use about 8 GB if I use 'qwen2.5-coder:7b' with 2 GB Swap
- The Disk Space you have that you can use
	- I have about 60 GB, 
		- I will use about 20 GB if I use 'qwen2.5-coder:3b'
		- I will use about 30 GB if I use 'qwen2.5-coder:7b'
- The Cores in CPU
	- I have about 12, I can use 8 without disturbing other processes.
		- I will use about 4 cores if I use 'qwen2.5-coder:3b'
		- I will use about 6 cores if I use 'qwen2.5-coder:7b'
- the CPU
- The GPU
- etc.

After you have an understanding, you can choose a model like *qwen2.5-coder:3b* and the chat application like *Aider*. I have made my choices earlier, you can choose something else if you so wish.

#### Create the Ollama LXC
Let us first create an LXC that will hold everything AI, I will proceed with 'qwen2.5-coder:3b' model here:
- name: `OllamaHost`
- LXC ID: `402`
- "Unprivileged container"
	- *Uncheck*: If you want to use GPU later or you want to avoid permission issues
	- *Check*: leave checked if you do not want the AI model to use GPU.
- password:
	- `qwen2.5-coder:3b`
- Template: ubuntu LXC
- Disks: 20 GB
- CPU 4 cores
- Memory: 8 GB and 2 GB swap
- Network
	- static IP: **`192.168.1.220`**
	- Gateway: `192.168.1.1`
- DNS
	- domain: `1.1.1.1` as my proxmox host always routes DNS with Netbird
	- server: `1.1.1.1` as my proxmox host always routes DNS with Netbird
- Proceed to complete

![075 Homelab Local AI LXC creation 01.png|100](/img/user/075%20Homelab%20Local%20AI%20LXC%20creation%2001.png)
![076 Homelab Local AI LXC creation 02.png|100](/img/user/076%20Homelab%20Local%20AI%20LXC%20creation%2002.png)
![077 Homelab Local AI LXC creation 03.png|100](/img/user/077%20Homelab%20Local%20AI%20LXC%20creation%2003.png)
![078 Homelab Local AI LXC creation 04.png|100](/img/user/078%20Homelab%20Local%20AI%20LXC%20creation%2004.png)
![079 Homelab Local AI LXC creation 05.png|100](/img/user/079%20Homelab%20Local%20AI%20LXC%20creation%2005.png)
![080 Homelab Local AI LXC creation 06.png|100](/img/user/080%20Homelab%20Local%20AI%20LXC%20creation%2006.png)
![081 Homelab Local AI LXC creation 07.png|100](/img/user/081%20Homelab%20Local%20AI%20LXC%20creation%2007.png)
![082 Homelab Local AI LXC creation 08 updated network.png|100](/img/user/082%20Homelab%20Local%20AI%20LXC%20creation%2008%20updated%20network.png)


```sh
[](https://www.proxmox.com/en/proxmox-virtual-environment/pricing)
()
Formatting '/mnt/games-setup/images/402/vm-402-disk-0.raw', fmt=raw size=21474836480 preallocation=off  
Creating filesystem with 5242880 4k blocks and 1310720 inodes  
Filesystem UUID: 55df1015-40da-4169-9279-7c1f2ff68316  
Superblock backups stored on blocks:  
32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,  
extracting archive '/var/lib/vz/template/cache/ubuntu-24.04-standard_24.04-2_amd64.tar.zst'  
Total bytes read: 564490240 (539MiB, 46MiB/s)  
Detected container architecture: amd64  
Setting up 'proxmox-regenerate-snakeoil.service' to regenerate snakeoil certificate..  
Creating SSH host key 'ssh_host_rsa_key' - this may take some time ...  
done: SHA256:AJY7wWLvwnuLIVyflxzwid/7y8tunSlDoPNFEYbr9yg root@OllamaHost  
Creating SSH host key 'ssh_host_ed25519_key' - this may take some time ...  
done: SHA256:0RFDXyyemrmEMcaWl44JAkpxzkSpPuS1w5eHwTgFXM0 root@OllamaHost  
Creating SSH host key 'ssh_host_ecdsa_key' - this may take some time ...  
done: SHA256:YANZq6+BE1KlcmoaTU/YkJZx8t2bhJXPETVh7Fg0EE4 root@OllamaHost  
WARN: Systemd 255 detected. You may need to enable nesting.  
TASK WARNINGS: 1
```

###### Enable nesting
This is optional, if the container runs without issue. See [[All Published Notes/Homelab/003 Proxmox - Ubuntu LXC as NAS with personal HDDs#Enable nesting\|Enable Nesting]].

#### Install Ollama
Log into the LXC and add/install `curl`, `zstd`, then get ollama from curl:
```sh
Sun Jul 12, 08:04:58 | root@pve:"~"# pct list
VMID       Status     Lock         Name                
400        running                 NAS-LXC             
401        running                 DockerHost          
402        stopped                 OllamaHost          
Sun Jul 12, 08:05:04 | root@pve:"~"# pct enter 402
container '402' not running!
Sun Jul 12, 08:05:21 | root@pve:"~"# pct start 402
WARN: Systemd 255 detected. You may need to enable nesting.
Task finished with 1 warning(s)!
Sun Jul 12, 08:05:30 | root@pve:"~"# pct enter 402
root@OllamaHost:~# pwd
/root
root@OllamaHost:~# ls /
bin  bin.usr-is-merged  boot  dev  etc  home  lib  lib.usr-is-merged  lib32  lib64  libx32  lost+found  media  mnt  opt  proc  root  run  sbin  sbin.usr-is-merged  srv  sys  tmp  usr  var
root@OllamaHost:~# ls
root@OllamaHost:~# 
root@OllamaHost:~# apt update && apt upgrade -y
...
done.
root@OllamaHost:~# 
root@OllamaHost:~# apt list -a curl
Listing... Done
curl/noble-updates,noble-security 8.5.0-2ubuntu10.11 amd64
curl/noble 8.5.0-2ubuntu10 amd64

root@OllamaHost:~# 
root@OllamaHost:~# apt install curl                                         
Reading package lists... Done
...

Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.7) ...
root@OllamaHost:~# curl -fsSL https://ollama.com/install.sh | sh
>>> Installing ollama to /usr
ERROR: This version requires zstd for extraction. Please install zstd and try again:
  - Debian/Ubuntu: sudo apt-get install zstd
  - RHEL/CentOS/Fedora: sudo dnf install zstd
  - Arch: sudo pacman -S zstd
root@OllamaHost:~# 
root@OllamaHost:~# apt install zstd
...
Setting up zstd (1.5.5+dfsg2-2build1.1) ...
Processing triggers for man-db (2.12.0-4build2) ...
root@OllamaHost:~# curl -fsSL https://ollama.com/install.sh | sh
>>> Cleaning up old version at /usr/lib/ollama
>>> Installing ollama to /usr
...
######################################################################## 100.0%
>>> The Ollama API is now available at 127.0.0.1:11434.
>>> Install complete. Run "ollama" from the command line.
>>> AMD GPU ready.
root@OllamaHost:~# 
```

Now, after `curl -fsSL https://ollama.com/install.sh | sh`, Ollama is part of your machine.

#### configure Ollama
Open the ollama service and allow systems outside the network to talk to it.
We can use `systemctl edit ollama.service` to edit the nano file, but it did not work for me, so I took an advice and created the `.conf` file with the desired config:
```sh
root@OllamaHost:~# mkdir -p /etc/systemd/system/ollama.service.d/
root@OllamaHost:~# cat <<EOF > /etc/systemd/system/ollama.service.d/override.conf
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
EOF
root@OllamaHost:~# 
root@OllamaHost:~# cat -n /etc/systemd/system/ollama.service.d/override.conf
     1  [Service]
     2  Environment="OLLAMA_HOST=0.0.0.0"
root@OllamaHost:~# 
```

Apply the changes:
```sh
root@OllamaHost:~# systemctl daemon-reload
root@OllamaHost:~# systemctl restart ollama
root@OllamaHost:~# 
```

To verify it worked, we look for '0.0.0.0:11434' in `ss -tlnp` output:
```sh
root@OllamaHost:~# ss -tlnp | grep 11434
LISTEN 0      4096               *:11434            *:*    users:(("ollama",pid=13152,fd=3))          
root@OllamaHost:~# 
```


### AI model to use
Use `ollama run qwen2.5-coder:3b-instruct` to download and run the model, we can also use `ollama run qwen2.5-coder:3b` to get the base model, but with `b-instruct` suffix we are picking the model that has been trained.

Also, if you want to get a model but not immediately start using it, you can use `ollama pull qwen2.5-coder:3b-instruct`.

```sh     
root@OllamaHost:~# ollama run qwen2.5-coder:3b-instruct
pulling manifest  
pulling 4a188102020e: 100% █████  1.9 GB                         
pulling 66b9ea09bd5b: 100% █████▏   68 B                         
pulling 1e65450c3067: 100% █████  1.6 KB                         
pulling 45fc3ea7579a: 100% █████▏ 7.4 KB                         
pulling bb967eff3bda: 100% █████   487 B                                               
verifying sha256 digest 
writing manifest 
success 
>>> hello
Hello! How can I assist you today?

>>> Send a message (/? for help)
```

Now that the model is installed, we can run basic tests before we move on to installing Aider on the development machine.
```sh

>>> hello
Hello! How can I assist you today?

>>> "Write a C# extension method for string reversal
Certainly! In C#, an extension method allows you to add new methods to existing types without modifying them directly. Below is an example of how you can create an extension method for reversing a `string`.

First, define the extension method within a static class:

```csharp
using System;

public static class StringExtensions
^C>>> 
>>> 
>>> 
>>> /?
Available Commands:
  /set            Set session variables
  /show           Show model information
  /load <model>   Load a session or model
  /save <model>   Save your current session
  /clear          Clear session context
  /bye            Exit
  /?, /help       Help for a command
  /? shortcuts    Help for keyboard shortcuts

Use """ to begin a multi-line message.

>>> /bye
root@OllamaHost:~# 
```

This took so long that I would be able to type it faster, so the model needs an upgrade.
Let's see the time a command takes by using verbose flag:
```sh

root@OllamaHost:~# ollama run --verbose qwen2.5-coder:3b
pulling manifest 
pulling 4a188102020e: 100% █████  1.9 GB                         
pulling 66b9ea09bd5b: 100% █████▏   68 B                         
pulling 1e65450c3067: 100% █████  1.6 KB                         
pulling 45fc3ea7579a: 100% █████▏ 7.4 KB                         
pulling bb967eff3bda: 100% █████   487 B                         
verifying sha256 digest 
writing manifest 
success 
>>> hello
Hello! How can I assist you today?

total duration:       31.824848017s
load duration:        149.446037ms
prompt eval count:    30 token(s)
prompt eval duration: 3.33283s
prompt eval rate:     9.00 tokens/s
eval count:           10 token(s)
eval duration:        28.340944s
eval rate:            0.35 tokens/s
>>> /bye
root@OllamaHost:~#
```
31 seconds for a hello, seems like I am at a rough spot.

For now, I will finish the setup, but will change model later.

We can try `export OLLAMA_NUM_PARALLEL=1` to force Ollama to limit CPU thread thrashing (Ollama will execute only one instruction at a time):
```sh
root@OllamaHost:~# export OLLAMA_NUM_PARALLEL=1                 
root@OllamaHost:~# systemctl restart ollama
root@OllamaHost:~# 
```


### Installing Aider on Development machine
We need to find a way to talk between our main development machine(Main PC called 'thegamelearner') in which we want to use AI on and the homelab.
We install Aider and it will be our communication point for our development machine, and instead of reaching to openAPI, we will tell it to go to our homelab's IP address instead.

Installing Aider can be done by running the official shell command(`curl -LsSf https://aider.chat/install.sh | sh`):
```sh
Tue Jul 14, 11:13:41 | thegamelearner@thegamelearner-MS-7E12:"~"
$ curl -LsSf https://aider.chat/install.sh | sh
downloading uv 0.5.9 x86_64-unknown-linux-gnu
no checksums to verify
installing to /home/thegamelearner/.local/bin
  uv
  uvx
uv is installed!
...
 + zipp==3.23.0
Installed 1 executable: aider
warning: `/home/thegamelearner/.local/bin` is not on your PATH. To use installed tools, run `export PATH="/home/thegamelearner/.local/bin:$PATH"` or `uv tool update-shell`.

To add $HOME/.local/bin to your PATH, either restart your shell or run:

    source $HOME/.local/bin/env (sh, bash, zsh)
    source $HOME/.local/bin/env.fish (fish)

Tue Jul 14, 11:14:17 | thegamelearner@thegamelearner-MS-7E12:"~"
$ 
```
To verify we can update the source and check the version:
```sh
Tue Jul 14, 11:14:17 | thegamelearner@thegamelearner-MS-7E12:"~"
$ aider --version
Command 'aider' not found, did you mean:
  command 'aide' from deb aide (0.18.6-2ubuntu0.1)
Try: sudo apt install <deb name>

Tue Jul 14, 11:15:54 | thegamelearner@thegamelearner-MS-7E12:"~"
$ source ~/.bashrc
[info] Currently loaded SSH keys:
256 SHA256:1y3WcQ3VM1PCjztudfrt2Mv/pta7TYyPkHMpmpC+WZ4 thegamelearner@gmail.com (ED25519)
3072 SHA256:UTu22gngWC/dmU+gJVDgkFLUvqoeMS4ThWyl0Gt0TrQ it.rishabh.jain@google.com (RSA)

Tue Jul 14, 11:15:59 | thegamelearner@thegamelearner-MS-7E12:"~"
$ aider --version
aider 0.86.2

Tue Jul 14, 11:16:02 | thegamelearner@thegamelearner-MS-7E12:"~"
$ 
```

#### Configuring Aider
We want aider to connect to our Ollama API base instead of searching Open API url, so we configure it's base to our IP and port:
```sh
export OLLAMA_API_BASE=http://192.168.1.220:11434
```
Here `192.168.1.220` is the IP address of the LXC where we installed Ollama with `qwen2.5-coder:3b-instruct` model.

Now, we can run qwen model through aider in main machine, but remember to run inside the git project rather than in open.

What happens when we run the model in home:
```sh
m -f .aider.input.history 

Tue Jul 14, 11:23:08 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ ls -al
total 24
drwxr-xr-x  3 thegamelearner thegamelearner  4096 Jul 14 23:23  .
drwxr-x--- 46 thegamelearner thegamelearner  4096 Jul 14 23:18  ..
-rwxrwxr-x  1 thegamelearner thegamelearner  1695 Oct  4  2025  addMp3Extension
drwxrwxr-x  5 thegamelearner thegamelearner 12288 Jul  2 23:35 'Music Library'

Tue Jul 14, 11:23:11 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ aider --model ollama/qwen2.5-coder:3b-instruct
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
No git repo found, create one to track aider's changes (recommended)? (Y)es/(N)o [Yes]: n                                                                                                                                                                                    
Aider v0.86.2
Model: ollama/qwen2.5-coder:3b-instruct with whole edit format
Git repo: none
Repo-map: disabled
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
> hello   
...

^C KeyboardInterrupt

Tue Jul 14, 11:30:22 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ ls -al
total 32
drwxr-xr-x  3 thegamelearner thegamelearner  4096 Jul 14 23:23  .
drwxr-x--- 46 thegamelearner thegamelearner  4096 Jul 14 23:18  ..
-rwxrwxr-x  1 thegamelearner thegamelearner  1695 Oct  4  2025  addMp3Extension
-rw-rw-r--  1 thegamelearner thegamelearner   817 Jul 14 23:30  .aider.chat.history.md
-rw-rw-r--  1 thegamelearner thegamelearner   138 Jul 14 23:29  .aider.input.history
drwxrwxr-x  5 thegamelearner thegamelearner 12288 Jul  2 23:35 'Music Library'

Tue Jul 14, 11:30:49 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ cat -n .aider.chat.history.md 
     1
     2  # aider chat started at 2026-07-14 23:23:20
     3
     4  > No git repo found, create one to track aider's changes (recommended)? (Y)es/(N)o [Yes]: n  
     5  > /home/thegamelearner/.local/bin/aider --model ollama/qwen2.5-coder:3b-instruct  
     6  > Aider v0.86.2  
     7  > Model: ollama/qwen2.5-coder:3b-instruct with whole edit format  
     8  > Git repo: none  
     9  > Repo-map: disabled  
    10
    11  #### hello  
    12
    13  To suggest changes to the greeting text to be more casual, I will:
    14
    15  1. Switch the greeting text from "Hello" to "Hey".
    16
    17  greeting.py
    18  ```
    19  import sys
    20
    21  def greeting(name):
    22      print(f"Hey {name}")
    23
    24  if __name__ == '__main__':
    25      greeting(sys.argv[1])
    26  ```
    27
    28  > Tokens: 592 sent, 66 received.  
    29  > greeting.py  
    30  > Create new file? (Y)es/(N)o [Yes]: n  
    31  > Skipping edits to greeting.py  
    32
    33  #### bye  
    34  >  
    35  >  
    36  >
{ #C}
 again to exit  
    37  >  
    38  >  
    39  >
{ #C}
 KeyboardInterrupt  

```

Now, we can use homelab as AI agent in our main machine.

### Configure Rider IDE

To use this model in Rider, open Rider's terminal and start aider same as before `aider --model ollama/qwen2.5-coder:3b-instruct`.
This gets the AI model running and you can use it as needed.

Once Aider launches inside Rider’s terminal, it will start monitoring your code workspace. To make sure it doesn't try to analyze your entire compilation directory or large binary blobs (which will stall your CPU), use these commands to select exactly what context it works on:
- **Adding Files to Context:** If you want to modify a specific class, type `/add Core/Services/UserService.cs`. Aider will pull that file path into active focus.
- **Architecture Mapping ("Ask" Mode):** If you want it to look at how files connect across the solution without opening them all, just ask it raw architecture questions: `/ask "Where is the database context registry initialized in our project entry point?"`
- **Agent Modification Mode ("Act" Mode):** Type out natural statements to make changes across your code: `"Refactor the UserService class to support async database fetching."`

All the best.

### Optional Deletion
#### How to delete a model

use `rm` to delete a model:
```sh
root@OllamaHost:~# ollama rm qwen2.5-coder:3b
deleted 'qwen2.5-coder:3b'
root@OllamaHost:~# ollama list               
NAME                         ID              SIZE      MODIFIED     
qwen2.5-coder:3b-instruct    f72c60cabf62    1.9 GB    47 hours ago    
root@OllamaHost:~# ollama rm qwen2.5-coder:3b-instruct
deleted 'qwen2.5-coder:3b-instruct'
root@OllamaHost:~# ollama list
NAME    ID    SIZE    MODIFIED 
root@OllamaHost:~# 
```

I cannot use the models, as 3b and 7b models take too long on my homelab, But the option is there if needed again.

#### How to delete aider
aider installs multiple dependencies so we need to remove them all:
```sh

Wed Jul 15, 12:06:11 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ rm -f ~/.local/bin/aider

Wed Jul 15, 12:09:05 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ rm -f ~/.local/bin/uv

Wed Jul 15, 12:09:09 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ rm -f ~/.local/bin/uvx

Wed Jul 15, 12:09:12 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ rm -rf ~/.local/share/aider

Wed Jul 15, 12:09:16 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ rm -rf ~/.local/share/uv

Wed Jul 15, 12:09:20 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ fresh ~/.bashrc

A new version of fresh is available: 0.2.17 -> 0.4.3
Update with: npm update -g @fresh-editor/fresh-editor


Wed Jul 15, 12:09:48 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ source ~/.bashrc
[info] Currently loaded SSH keys:
256 SHA256:1y3WcQ3VM1PCjztudfrt2Mv/pta7TYyPkHMpmpC+WZ4 thegamelearner@gmail.com (ED25519)
3072 SHA256:UTu22gngWC/dmU+gJVDgkFLUvqoeMS4ThWyl0Gt0TrQ it.rishabh.jain@google.com (RSA)

Wed Jul 15, 12:10:01 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ aider --version
Command 'aider' not found, did you mean:
  command 'aide' from deb aide (0.18.6-2ubuntu0.1)
Try: sudo apt install <deb name>

Wed Jul 15, 12:10:06 | thegamelearner@thegamelearner-MS-7E12:"~/Music"
$ 
```


### Install AI on main machine
On main machine, I have more GPU and CPU, so I will try to install a model on this:
```sh
Wed Jul 15, 12:22:16 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/AI_Models"
$ pwd
/home/thegamelearner/Documents/AI_Models

Wed Jul 15, 12:22:18 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/AI_Models"
$ docker run -d \
  --device=/dev/kfd \
  --device=/dev/dri \
  --cpus="12" \
  --memory="16g" \
  -v ollama:/root/.ollama \
  -p 11434:11434 \
  --name ollama \
  ollama/ollama:rocm
.Command 'docker' not found, but can be installed with:
sudo apt install docker.io      # version 29.1.3-0ubuntu3~24.04.2, or
sudo apt install podman-docker  # version 4.9.3+ds1-1ubuntu0.2

Wed Jul 15, 12:22:22 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/AI_Models"
$ sudo apt install docker.io
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
...
Setting up docker.io (29.1.3-0ubuntu3~24.04.2) ...
info: Selecting GID from range 100 to 999 ...
info: Adding group `docker` (GID 128) ...
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /usr/lib/systemd/system/docker.socket.
Processing triggers for man-db (2.12.0-4build2) ...

Wed Jul 15, 12:23:04 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/AI_Models"
$ docker run -d   --device=/dev/kfd   --device=/dev/dri   --cpus="12"   --memory="16g"   -v ollama:/root/.ollama   -p 11434:11434   --name ollama   ollama/ollama:rocm
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock

Wed Jul 15, 12:23:35 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/AI_Models"
$ sudo docker run -d   --device=/dev/kfd   --device=/dev/dri   --cpus="12"   --memory="16g"   -v ollama:/root/.ollama   -p 11434:11434   --name ollama   ollama/ollama:rocm
Unable to find image 'ollama/ollama:rocm' locally
rocm: Pulling from ollama/ollama
ca2678b20700: Pull complete 
f0178b7ea30b: Pull complete 
1eb256f139a3: Pull complete 
a1cdce1862ea: Pull complete 
Digest: sha256:ed3ff2d663fba3b089807a8dca022af9fc1870bbcb7ed4bba9ce5f3939821269
Status: Downloaded newer image for ollama/ollama:rocm
d1bbfa674b71438c5ceea0f67121a721e060f4e715ff03bbacb9c2f1921285a5

Wed Jul 15, 12:29:51 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/AI_Models"
$ sudo docker exec -it ollama ollama run llama3.1
pulling manifest 
pulling 667b0c1932bc: 100% ▕███████████████████▏ 4.9 GB                         
pulling 948af2743fc7: 100% ▕███████████████████▏ 1.5 KB                         
pulling 0ba8f0e314b4: 100% ▕███████████████████▏  12 KB                         
pulling 56bb8bd477a5: 100% ▕███████████████████▏   96 B                         
pulling 455f34728c9b: 100% ▕███████████████████▏  487 B                         
verifying sha256 digest 
writing manifest 
success 
>>> /bye

Wed Jul 15, 12:53:37 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/AI_Models"
$ docker exec -it ollama ollama run llama3.1 --verbose
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock

Wed Jul 15, 12:53:39 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/AI_Models"
$ sudo docker exec -it ollama ollama run llama3.1 --verbose
[sudo] password for thegamelearner:           
>>> hello
Hello! How can I assist you today?

total duration:       295.877148ms
load duration:        142.825897ms
prompt eval count:    11 token(s)
prompt eval duration: 36.226ms
prompt eval rate:     303.65 tokens/s
eval count:           10 token(s)
eval duration:        115.317ms
eval rate:            86.72 tokens/s
>>> write a c# extension code to reverse a string
Here is an example of a C# extension method to reverse a string:
``csharp
public static class StringExtensions
{
    /// <summary>
    /// Reverses the input string.
    /// </summary>
    /// <param name="value">The input string.</param>
    /// <returns>The reversed string.</returns>
    public static string Reverse(this string value)
    {
        if (string.IsNullOrEmpty(value))
            return value;

        char[] arr = value.ToCharArray();
        Array.Reverse(arr);
        return new string(arr);
    }
}
``
You can use this extension method like this:
``csharp
string originalString = "hello";
string reversedString = originalString.Reverse();

Console.WriteLine(reversedString); // Output: olleh
``
Note that this implementation uses the `Array.Reverse` method to reverse the characters in the string, which is a more efficient approach than using LINQ.

Also, as with any extension method, you need to add the namespace where it`s defined to your using directives for it to work. For example:
``csharp
using MyNamespace.StringExtensions;
``
Replace `MyNamespace` with the actual namespace where you`ve defined this extension method.

total duration:       3.279845275s
load duration:        143.311098ms
prompt eval count:    40 token(s)
prompt eval duration: 35.775ms
prompt eval rate:     1118.10 tokens/s
eval count:           248 token(s)
eval duration:        3.099775s
eval rate:            80.01 tokens/s
>>> /bye

Wed Jul 15, 12:54:43 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/AI_Models"
$
{ #C}


Wed Jul 15, 12:54:57 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/AI_Models"
$ 
```

This returns a reply to 'hello' in 295.877148 ms.
This serves the need completely.

Important commands:
- `sudo docker exec -it ollama ollama run llama3.1 --verbose` to run `llama3.1` model in `verbose` mode.
- Inside ollama CLI
	- `Ctrl + a`: Move to the beginning of the line (Home)
	- `Ctrl + e`: Move to the end of the line (End)
	- `Alt + b`: Move back (left) one word
	- `Alt + f`: Move forward (right) one word
	- `Ctrl + k`: Delete the sentence after the cursor
	- `Ctrl + u`: Delete the sentence before the cursor
	- `Ctrl + w`: Delete the word before the cursor
	- `Ctrl + l`: Clear the screen
	- `Ctrl + g`: Open default editor to compose a prompt
	- `Ctrl + c`: Stop the model from responding
	- `Ctrl + d`: Exit ollama (/bye)

![083 Homelab Local AI Rider Setting.png](/img/user/083%20Homelab%20Local%20AI%20Rider%20Setting.png)


---

[^1]: 
[^2]: 

