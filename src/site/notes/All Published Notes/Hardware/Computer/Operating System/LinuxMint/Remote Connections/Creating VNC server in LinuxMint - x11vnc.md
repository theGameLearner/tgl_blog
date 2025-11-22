---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/operating-system/linux-mint/remote-connections/creating-vnc-server-in-linux-mint-x11vnc/"}
---

created: 2025-08-31
updated: 2025-11-22

### Theory
We are using x11vnc for server and you can use the viewer of your choice. We want to share the same session so that we can change to remote machine and continue from where we were working without missing a beat.

### Check if VNC server of choice is installed
Check if the machine to connect to (server) has x11vnc installed:
- `sudo apt list --installed | grep vnc`
- download if you have not done so: `sudo apt install x11vnc`

`lightdm` - LightDM is the display manager, responsible for providing the graphical login screen (greeter) and managing the graphical user sessions after you log in. This is pre-installed in Linux mint, if you are using another Ubuntu based distribution, check if it is installed. If it is not installed, ensure you can configure it or get something like it on your distribution.

### Start the server

To start x11vnc, use the following command:
```sh
x11vnc -display :0 -geometry 1920x1080 -depth 24 # geometry defines the resolution and depth defines the color (OR)
x11vnc -display :0 # <<< USE THIS COMMAND ! to ensure the same display is shared
```
- `x11vnc -display :0` this ensures that we share the current display (Shares the current X session)
- `x11vnc -display :0 -geometry 1920x1080 -depth 24`
	- geometry is used to define the resolution(you can change it to one that is supported by your viewer), 
	- depth is used to define the color (Ensures 24-bit color depth for better color quality.)
get the ip-address of the server:
```sh
thegamelearner@thegamelearner-MS-7E12:~$ ifconfig
enp7s0: flags=4163  mtu 1500
        inet 192.168.31.204  netmask 255.255.255.0  broadcast 192.168.31.255
        ...
```
as both machines are connected to same Wi-Fi, we need the `enp7s0` IP address.

#### Set the password
Password is not necessary for accessing, but a preferred choice. You can use a custom file to set the password or the default path `~/.vnc/passwd` will be used. I will skip this as I have not determined the performance of the connection.
#### warnings
You can skip the next terminal. It is all warnings and error you can get when you start the server:
```sh title:complete_warnings
thegamelearner@thegamelearner-MS-7E12:~$ x11vnc -display :0
###############################################################
#@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@#
#@                                                           @#
#@  **  WARNING  **  WARNING  **  WARNING  **  WARNING  **   @#
#@                                                           @#
#@        YOU ARE RUNNING X11VNC WITHOUT A PASSWORD!!        @#
#@                                                           @#
#@  This means anyone with network access to this computer   @#
#@  may be able to view and control your desktop.            @#
#@                                                           @#
#@ >>> If you did not mean to do this Press CTRL-C now!! <<< @#
#@                                                           @#
#@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@#
#@                                                           @#
#@  You can create an x11vnc password file by running:       @#
#@                                                           @#
#@       x11vnc -storepasswd password /path/to/passfile      @#
#@  or   x11vnc -storepasswd /path/to/passfile               @#
#@  or   x11vnc -storepasswd                                 @#
#@                                                           @#
#@  (the last one will use ~/.vnc/passwd)                    @#
#@                                                           @#
#@  and then starting x11vnc via:                            @#
#@                                                           @#
#@      x11vnc -rfbauth /path/to/passfile                    @#
#@                                                           @#
#@  an existing ~/.vnc/passwd file from another VNC          @#
#@  application will work fine too.                          @#
#@                                                           @#
#@  You can also use the -passwdfile or -passwd options.     @#
#@  (note -passwd is unsafe if local users are not trusted)  @#
#@                                                           @#
#@  Make sure any -rfbauth and -passwdfile password files    @#
#@  cannot be read by untrusted users.                       @#
#@                                                           @#
#@  Use x11vnc -usepw to automatically use your              @#
#@  ~/.vnc/passwd or ~/.vnc/passwdfile password files.       @#
#@  (and prompt you to create ~/.vnc/passwd if neither       @#
#@  file exists.)  Under -usepw, x11vnc will exit if it      @#
#@  cannot find a password to use.                           @#
#@                                                           @#
#@                                                           @#
#@  Even with a password, the subsequent VNC traffic is      @#
#@  sent in the clear.  Consider tunnelling via ssh(1):      @#
#@                                                           @#
#@    http://www.karlrunge.com/x11vnc/#tunnelling            @#
#@                                                           @#
#@  Or using the x11vnc SSL options: -ssl and -stunnel       @#
#@                                                           @#
#@  Please Read the documentation for more info about        @#
#@  passwords, security, and encryption.                     @#
#@                                                           @#
#@    http://www.karlrunge.com/x11vnc/faq.html#faq-passwd    @#
#@                                                           @#
#@  To disable this warning use the -nopw option, or put     @#
#@  'nopw' on a line in your ~/.x11vncrc file.               @#
#@                                                           @#
#@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@#
###############################################################
31/08/2025 14:32:32 x11vnc version: 0.9.16 lastmod: 2019-01-05  pid: 26784
31/08/2025 14:32:32 Using X display :0
31/08/2025 14:32:32 rootwin: 0x40f reswin: 0x7800001 dpy: 0x1a2aeee0
31/08/2025 14:32:32 
31/08/2025 14:32:32 ------------------ USEFUL INFORMATION ------------------
31/08/2025 14:32:32 X DAMAGE available on display, using it for polling hints.
31/08/2025 14:32:32   To disable this behavior use: '-noxdamage'
31/08/2025 14:32:32 
31/08/2025 14:32:32   Most compositing window managers like 'compiz' or 'beryl'
31/08/2025 14:32:32   cause X DAMAGE to fail, and so you may not see any screen
31/08/2025 14:32:32   updates via VNC.  Either disable 'compiz' (recommended) or
31/08/2025 14:32:32   supply the x11vnc '-noxdamage' command line option.
31/08/2025 14:32:32 
31/08/2025 14:32:32 Wireframing: -wireframe mode is in effect for window moves.
31/08/2025 14:32:32   If this yields undesired behavior (poor response, painting
31/08/2025 14:32:32   errors, etc) it may be disabled:
31/08/2025 14:32:32    - use '-nowf' to disable wireframing completely.
31/08/2025 14:32:32    - use '-nowcr' to disable the Copy Rectangle after the
31/08/2025 14:32:32      moved window is released in the new position.
31/08/2025 14:32:32   Also see the -help entry for tuning parameters.
31/08/2025 14:32:32   You can press 3 Alt_L's (Left "Alt" key) in a row to 
31/08/2025 14:32:32   repaint the screen, also see the -fixscreen option for
31/08/2025 14:32:32   periodic repaints.
31/08/2025 14:32:32 
31/08/2025 14:32:32 XFIXES available on display, resetting cursor mode
31/08/2025 14:32:32   to: '-cursor most'.
31/08/2025 14:32:32   to disable this behavior use: '-cursor arrow'
31/08/2025 14:32:32   or '-noxfixes'.
31/08/2025 14:32:32 using XFIXES for cursor drawing.
31/08/2025 14:32:32 GrabServer control via XTEST.
31/08/2025 14:32:32 
31/08/2025 14:32:32 Scroll Detection: -scrollcopyrect mode is in effect to
31/08/2025 14:32:32   use RECORD extension to try to detect scrolling windows
31/08/2025 14:32:32   (induced by either user keystroke or mouse input).
31/08/2025 14:32:32   If this yields undesired behavior (poor response, painting
31/08/2025 14:32:32   errors, etc) it may be disabled via: '-noscr'
31/08/2025 14:32:32   Also see the -help entry for tuning parameters.
31/08/2025 14:32:32   You can press 3 Alt_L's (Left "Alt" key) in a row to 
31/08/2025 14:32:32   repaint the screen, also see the -fixscreen option for
31/08/2025 14:32:32   periodic repaints.
31/08/2025 14:32:32 
31/08/2025 14:32:32 XKEYBOARD: number of keysyms per keycode 7 is greater
31/08/2025 14:32:32   than 4 and 52 keysyms are mapped above 4.
31/08/2025 14:32:32   Automatically switching to -xkb mode.
31/08/2025 14:32:32   If this makes the key mapping worse you can
31/08/2025 14:32:32   disable it with the "-noxkb" option.
31/08/2025 14:32:32   Also, remember "-remap DEAD" for accenting characters.
31/08/2025 14:32:32 
31/08/2025 14:32:32 X FBPM extension not supported.
31/08/2025 14:32:32 X display is capable of DPMS.
31/08/2025 14:32:32 --------------------------------------------------------
31/08/2025 14:32:32 
31/08/2025 14:32:32 Default visual ID: 0x21
31/08/2025 14:32:32 Read initial data from X display into framebuffer.
31/08/2025 14:32:32 initialize_screen: fb_depth/fb_bpp/fb_Bpl 24/32/17920
31/08/2025 14:32:32 
31/08/2025 14:32:32 X display :0 is 32bpp depth=24 true color
31/08/2025 14:32:32 
31/08/2025 14:32:32 Autoprobing TCP port 
31/08/2025 14:32:32 Autoprobing selected TCP port 5900
31/08/2025 14:32:32 Autoprobing TCP6 port 
31/08/2025 14:32:32 Autoprobing selected TCP6 port 5900
31/08/2025 14:32:32 listen6: bind: Address already in use
31/08/2025 14:32:32 Not listening on IPv6 interface.
31/08/2025 14:32:32 
31/08/2025 14:32:32 Xinerama is present and active (e.g. multi-head).
31/08/2025 14:32:32 Xinerama: number of sub-screens: 2
31/08/2025 14:32:32 Xinerama: enabling -xwarppointer mode to try to correct
31/08/2025 14:32:32 Xinerama: mouse pointer motion. XTEST+XINERAMA bug.
31/08/2025 14:32:32 Xinerama: Use -noxwarppointer to force XTEST.
31/08/2025 14:32:32 Xinerama: sub-screen[0]  2560x1440+0+0
31/08/2025 14:32:32 Xinerama: sub-screen[1]  1920x1080+2560+360
31/08/2025 14:32:32 blackout rect: 1920x360+2560+0: x=2560-4479 y=0-360
31/08/2025 14:32:32 
31/08/2025 14:32:32 fb read rate: 412 MB/sec
31/08/2025 14:32:32 fast read: reset -wait  ms to: 10
31/08/2025 14:32:32 fast read: reset -defer ms to: 10
31/08/2025 14:32:32 The X server says there are 10 mouse buttons.
31/08/2025 14:32:32 screen setup finished.
31/08/2025 14:32:32 
31/08/2025 14:32:32 WARNING: You are running x11vnc WITHOUT a password.  See
31/08/2025 14:32:32 WARNING: the warning message printed above for more info.
31/08/2025 14:32:32 

The VNC desktop is:      thegamelearner-MS-7E12:0
PORT=5900

******************************************************************************
Have you tried the x11vnc '-ncache' VNC client-side pixel caching feature yet?

The scheme stores pixel data offscreen on the VNC viewer side for faster
retrieval.  It should work with any VNC viewer.  Try it by running:

    x11vnc -ncache 10 ...

One can also add -ncache_cr for smooth 'copyrect' window motion.
More info: http://www.karlrunge.com/x11vnc/faq.html#faq-client-caching
```

#### Allow others to connect on a port
Update the 'ufw' in the server to allow others to connect on the port '5900' which is the default port for x11vnc server.
firewall status : `sudo ufw status`
allowing new port: `sudo ufw allow 5900/tcp`
deleting any rule: `sudo ufw delete allow 5900/tcp`
reload rules: `sudo ufw reload` - do this if the rules are not updated.
```sh
thegamelearner@thegamelearner-MS-7E12:~$ sudo ufw status
[sudo] password for thegamelearner:           
Status: active
thegamelearner@thegamelearner-MS-7E12:~$ sudo ufw status
Status: active
thegamelearner@thegamelearner-MS-7E12:~$ sudo ufw allow 5900/tcp
Rule added
Rule added (v6)
thegamelearner@thegamelearner-MS-7E12:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
5900/tcp                   ALLOW       Anywhere                  
5900/tcp (v6)              ALLOW       Anywhere (v6)             

thegamelearner@thegamelearner-MS-7E12:~$ 
```


### Connecting to the server
In your remote viewer machine, you need a viewer app to view the server machine. As we are using VNC, we can use any viewer, I will use TightVnc.
Install TightVnc-viewer:
```sh
$ sudo apt install tigervnc-viewer
```
Start the viewer:
```sh
vncviewer
```
you will get a pop up asking for the IP address, enter the IP address of the server, in our example `192.168.31.204:5900`. The port '5900' is the default port.

My chromebook has the resolution 1366x768(Native), so we will use this resolution when starting the x11vnc server: `x11vnc -display :0 -geometry 1366x768`

### Summary
The connection was successful, the display was lagging a little, but this can be because my server is powerful while my chromebook is not. The remote chromebook showed both my monitors connected to the main server and I could even operate and see on server monitor what i am doing.
Audio was not shared, as I was able to play youtube on my server but my remote only showed the screen without the audio.
This option can be considered a good choice if you only need the screen for a rush work.



---

[^1]: github page: https://github.com/LibVNC/x11vnc
[^2]: forums: https://forums.linuxmint.com/viewtopic.php?t=440131
[^3]: youtube: https://www.youtube.com/watch?v=BQnuH6DnPoo



