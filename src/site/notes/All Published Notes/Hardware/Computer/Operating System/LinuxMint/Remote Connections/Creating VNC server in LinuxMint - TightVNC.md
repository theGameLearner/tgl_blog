---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/operating-system/linux-mint/remote-connections/creating-vnc-server-in-linux-mint-tight-vnc/"}
---

created: 2025-08-27
updated: 2025-11-22
### Theory
We are using [TightVNC](https://www.tightvnc.com/) for server and viewer. This by default allows us to VNC into another machine with a new session, meaning the local machine can have one user logged in and using the machine, and you can log in as the same user or a separate user on same machine. Think thin client connections on a server for better understanding.
To use x11vnc, so that we can take over existing monitor and session, you can see [[All Published Notes/Hardware/Computer/Operating System/LinuxMint/Remote Connections/Creating VNC server in LinuxMint - x11vnc\|Creating VNC server in LinuxMint - x11vnc]]
### Check if VNC server of choice is installed
Check if the machine to connect to (server) has tightvnc installed:
- `sudo apt list --installed | grep vnc`
- [download](https://www.tightvnc.com/download.php) if you have not done so: `sudo apt install tightvncserver`

### Start the server
Start the server and set the password in the machine:
- use the command `vncserver` to create a VNC server instance
	- You will be prompted for entering a password for read and write `rishabhJ`
	- You will get a choice to enter a view only password `12345678`
	- The password must be between six and eight characters long
- use command `vncpasswd` to change password if you want to change it
- The server can be stopped by killing the process `vncserver -kill :1` where the number `1` is the port identifier in use or the display port number, because of line: `New 'X' desktop is thegamelearner-MS-7E12:1`, meaning we are using port 5901(5900+display port number).

```sh
thegamelearner@thegamelearner-MS-7E12:~$ vncserver

You will require a password to access your desktops.

Password: rishabhJ
Warning: password truncated to the length of 8.
Verify:   rishabhJ
Would you like to enter a view-only password (y/n)? y
Password: 12345678
Warning: password truncated to the length of 8.
Verify:   12345678

New 'X' desktop is thegamelearner-MS-7E12:1

Creating default startup script /home/thegamelearner/.vnc/xstartup
Starting applications specified in /home/thegamelearner/.vnc/xstartup
Log file is /home/thegamelearner/.vnc/thegamelearner-MS-7E12:1.log

thegamelearner@thegamelearner-MS-7E12:~$ 
thegamelearner@thegamelearner-MS-7E12:~$ vncpasswd
Using password file /home/thegamelearner/.vnc/passwd
Password: 
Verify:   
Would you like to enter a view-only password (y/n)? y
Password: 
Verify:   
thegamelearner@thegamelearner-MS-7E12:~$ 
thegamelearner@thegamelearner-MS-7E12:~$ vncserver -kill :1
Killing Xtightvnc process ID 82188
thegamelearner@thegamelearner-MS-7E12:~$
```

### Configurations of the server
The server configurations are set in `/home/username/.vnc/xstartup` or `~/.vnc/xstartup` file. We can take a backup if we want to make changes in it.
Original code:
```sh
thegamelearner@thegamelearner-MS-7E12:~$ cat ~/.vnc/xstartup
#!/bin/sh
xrdb "$HOME/.Xresources"
xsetroot -solid grey
#x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
#x-window-manager &
# Fix to make GNOME work
export XKL_XMODMAP_DISABLE=1
/etc/X11/Xsession
thegamelearner@thegamelearner-MS-7E12:~$
```
Now, we can either install a Desktop environment if you are working with a server or we can use existing environment. I have a cinnamon desktop environment with Linux mint, so we want to use it, I am using the following to identify which Desktop environment I have:
```sh title:identifying_Desktop_enviroinment
thegamelearner@thegamelearner-MS-7E12:~$ inxi -S
System:
  Host: thegamelearner-MS-7E12 Kernel: 6.8.0-56-generic arch: x86_64 bits: 64
  Desktop: Cinnamon v: 6.4.8 Distro: Linux Mint 22.1 Xia
thegamelearner@thegamelearner-MS-7E12:~$ echo $XDG_SESSION_DESKTOP
cinnamon
thegamelearner@thegamelearner-MS-7E12:~$ echo $XDG_CURRENT_DESKTOP
X-Cinnamon
thegamelearner@thegamelearner-MS-7E12:~$ echo $cinnamon-session-cinnamon
-session-cinnamon
thegamelearner@thegamelearner-MS-7E12:~$
```

Using the info I fetched before, we will set the export variables within our VNC environment so it matches what we have, update the `~/.vnc/xstartup` file to:

```sh title:~/.vnc/xstartup
# ~/.vnc/xstartup
# !/bin/sh
xrdb "$HOME/.Xresources" # for loading X resources
export XKL_XMODMAP_DISABLE=1 # this is important for keyboard mapping issues, where old system setting and new system setting conflicts can be ignored
export XDG_CURRENT_DESKTOP="X-Cinnamon" # This helps set the current desktop setup as "X-Cinnamon" which was already available with a GUI compatible system
export XDG_SESSION_DESKTOP="cinnamon" # This helps set the current desktop session's setup as "cinnamon" which was already available with a GUI compatible system
dbus-launch cinnamon-session --session cinnamon & 
# `dbus-launch` ensures proper D-Bus session management
# `cinnamon-session --session cinnamon` explicitly specifies and starts the Cinnamon session
# `&` ensures this runs in BG, so current user is not effected by the new session
```

The first command in the file, `xrdb $HOME/.Xresources`, tells VNC’s GUI framework to read the server user’s `.Xresources` file. `.Xresources` is where a user can make changes to certain settings of the graphical desktop, like terminal colors, cursor themes, and font rendering. The second command tells to prioritize keyboard mapping based on **X Keyboard Extension (XKB)** for keyboard configuration. The new VNC does not have environmental variables from the existing machine, so we export the variables we want it to have, in our case '*XDG_CURRENT_DESKTOP*' and '*XDG_SESSION_DESKTOP*'.
Finally, we launch the cinnamon session in background, we include dbus, as it is essential for cinnamon.

### Connecting to the server
After updating the settings, we can try to use it.
Verify the file is executable (has x):
```sh
thegamelearner@thegamelearner-MS-7E12:~$ ls -al ~/.vnc/xstartup
-rwxr-xr-x 1 thegamelearner thegamelearner 1359 Aug 30 23:38 /home/thegamelearner/.vnc/xstartup
thegamelearner@thegamelearner-MS-7E12:~$
```
and that the server is running:
```sh
ps aux | grep vnc
```
We have 2 ways to connect now, depending on your use case:
- [[#SSH Tunnelling and using VNC]] - this is used when you want to become a user on the server machine and use SSH's encryption to keep your work safe from prying eyes.
- [[#Connecting over LAN]] - This is useful in local area network like a home where you know and trust every device connected to same WiFi or Ethernet connection.

In case you want to change the port in use, use 
#### SSH Tunnelling and using VNC
If you plan to use SSH tunnelling, you can start the server with `vncserver -localhost` instead of `vncserver` to ensure it can be connected only from the same machine. What it will achieve is that you connect via SSH tunnelling to your VNC server and then you start VNC as if you are already on the server machine. making the connection local to the server machine.

Make a SSH server on your server machine, and then connect to the server using SSH tunneling from your client machine.

We want to run VNC as a service so we can *start*, *stop*, and *restart* it as needed. 
##### creating the VNC service
create a new unit file called `/etc/systemd/system/vncserver@.service`:
```sh
sudo nano /etc/systemd/system/vncserver@.service
```
The `@` symbol at the end of the name will let us pass in an argument you can use in the service configuration. You’ll use this to specify the VNC display port you want to use when you manage the service.

Add the following lines to the file. Be sure to change the value of **User**, **Group**, **WorkingDirectory**, and the username in the value of **PIDFILE** to match your username:
```sh title:/etc/systemd/system/vncserver@.service
[Unit]
Description=Start TightVNC server at startup
After=syslog.target network.target

[Service]
Type=forking
User=sammy
Group=sammy
WorkingDirectory=/home/sammy

PIDFile=/home/sammy/.vnc/%H:%i.pid
ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 -localhost :%i
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target
```
The `ExecStartPre` command stops VNC if it’s already running. The `ExecStart` command starts VNC and sets the color depth to 24-bit color with a resolution of 1280x800. You can modify these startup options as well to meet your needs. Also, note that the `ExecStart` command again includes the `-localhost` option.

Next, make the system aware of the new unit file:
```sh
sudo systemctl daemon-reload
```
enable this service:
```sh
sudo systemctl enable vncserver@1.service
```
in above command `@1` represents port 1 of VNC server(5901).
start the server:
```sh
sudo systemctl start vncserver@1
```
verify that it started with this command:
```sh
sudo systemctl status vncserver@1
```

#### Connecting over LAN (Viewer)

Install a viewer on the device to use as the client, I am using `sudo apt install tigervnc-viewer` (as consistency is important) on my chromebook. 

run `vncviewer` to connect
you get a pop up asking for the VNC server:`192.168.31.204:5901`
hit `connect`.
you get a pop up with password, 
- if you choose the password for view only, you cannot click on anything or do anything (`12345678`)
- if you enter the password for read and write, you can work on it as a normal machine.(`rishabhJ`)
	- remember you start a new session, so the screen is not shared and there is no way to know what is happening, except for the VNC session.
> [!Note]
> If you are not asked for password, a common issue is the firewall blocks port `590n`. Just open this port in firewall settings based on which firewall you use and try again.

firewall status : `sudo ufw status`
allowing new port: `sudo ufw allow 5901/tcp`
deleting any rule: `sudo ufw delete allow 5901/tcp`
reload rules: `sudo ufw reload`


### Summary

The direct connection(without SSH Tunnelling), though easy to setup, was lagging in performance. It can be because my same user session was active on the server machine. But when my server is powerful and quick, i would expect my client to reflect the power of my server.

---

[^1]: [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-20-04)
[^2]: [ionos](https://www.ionos.com/digitalguide/server/configuration/vnc-server-ubuntu-2204/)
