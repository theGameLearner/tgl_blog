---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/operating-system/connecting-remotely/"}
---

created: 2025-08-27
updated: 2025-11-22
### Requirements
To follow this theory, you will need:
- A machine which is up and running which we want to access with remote connection.
- Another machine which you will use to access the main machine. (can use mobile phone as well)
- a Wi-Fi or LAN connection shared by the two machines
	- There are ways to connect when the two systems are not connected to same WI-Fi or to each other by LAN, but that is not the focus for now.
### Scenario
You have a system in a room, you have reasons (bed rest, system has too many peripherals attached and you are in another place with guests, etc) that you cannot directly or physically access the system, but you want to access files in the system or do things on it.
Assuming you have another system like a phone or Chromebook (ChromeOS), which is light weight, you can access your main system using multiple solutions:

### Protocols
There are many protocols that allow one system to connect to another and use it as if you are sitting on the other system:
- ***GUI based access protocols***: We see the desktop and can use mouse and keyboard to interact.
	- **RFB (Remote Frame Buffer) / VNC (Virtual Network Computing)** : The underlying protocol for VNC (Virtual Network Computing). It transmits the screen as a series of pixel updates ("screenshots"). It is cross-platform but bandwidth-heavy.
		- when you have different Operating systems and want to share the mouse and keyboard actions without issues.
		- Session Logic:
			- *Shared (Standard)*: The remote user sees exactly what is on the physical monitor. Both the local and remote users fight for control of the mouse. (e.g., `x11vnc`).
			- *New (Linux specific):* Some implementations (like `TightVNC` on Linux) create a virtual desktop in the background. The physical monitor shows one thing, while the remote user works on a completely different, invisible desktop.
				- here a new user session is created where you interact with the machine, so you may consider it as opening the machine from a new location as a different user, and any active session is not copied over.
	- **Remote Desktop Protocol (RDP)** : Developed by Microsoft, 'RDP' is the standard for remote connections to *Windows* machines. 
		- It is not natively used in other OS, but with `xrdp` tool, we can make Linux an RDP server and use with it.
			- `Remmina` Client app (Linux) and the official "Microsoft Remote Desktop" app (macOS/Android/iOS) make RDP native-feeling on all platforms.
		- If you use RDP to connect, you are taking over the existing session, the user's session is carried forward by you with no change.
		- Session Logic:
			- *Authority (Default on Windows Client):* When you connect to a Windows PC (e.g., Windows 10 Pro), the local screen locks. The remote user has exclusive control; the local user cannot see or interact without kicking the remote user off.
			- *New (Windows Server / Linux xrdp):* On server operating systems, 'RDP' can create a fresh, independent desktop instance for the remote user while other users continue working undisturbed.
	- **NX (NoMachine / NX Protocol)** : A highly compressed protocol designed for Linux to make X11 graphical interfaces fast over slow networks. It handles audio and device sharing better than raw VNC.
		- High-Performance / Proprietary(compressed X11) protocol.
		- Session Logic:
			- *New:* Typically creates a new virtual session on the Linux server.
			- *Shared:* Can be configured to "Shadow" an existing physical display, allowing for collaborative viewing.
	- **SPICE (Simple Protocol for Independent Computing Environments)** : Designed specifically for Virtual Machines (like QEMU/KVM). It provides high-quality video and audio streaming from a VM to a client.
		- Session Logic:
			- *Shared:* It acts as the "Console" for the Virtual Machine. Connecting via SPICE is equivalent to standing physically in front of the VM's monitor.
- ***Command-line Interface***: We talk using terminal commands, we do not see the GUI screen.
	- **Secure Shell (SSH)** : The standard cryptographic network protocol for operating network services securely over an unsecured network.
		- Used to securely connect to a device's command line, we cannot see the graphical user interface(GUI) but can connect to the terminal and run terminal commands through the shell.
		- you login as a user on the device with access to the terminal, the changes you make on the system is same as the same changes made by the same user directly working on the server machine.
		- Session Logic:
			- *New:* Every SSH connection spawns a 'new shell instance'. You are logged in as a specific user, but your terminal actions are independent of any other open terminals or the GUI.
	- **Mosh (Mobile Shell)** : A protocol built on top of 'UDP' (unlike SSH which uses 'TCP') designed for roaming and unstable connections. It allows the client to roam between IP addresses without dropping the connection.
		- Session Logic:
			- *New:* Like SSH, it creates a new independent shell session.
	- **Telnet** : A predecessor to SSH, Telnet is a basic, 'unencrypted' protocol for command-line access. It can be ignored for most use cases. as creating unsecure connection is always risky.
		- Session Logic:
			- *New:* Creates a new shell session, but transmits all data (including passwords) in plain text.
- ***Hybrid / Application Access*** : Protocols that do not send a "Desktop" but rather specific windows.
	- **X11 Forwarding (via SSH)** : Allows a Linux GUI application running on a remote server to display its window on the local client machine's screen, transported through an encrypted SSH tunnel.
		- Session Logic:
			- *Integrated:* There is no "Remote Desktop." The remote app window appears alongside your local apps. The logic is tied to the SSH user session (New).
	- **Waypipe (Wayland Forwarding)** : The modern equivalent of X11 Forwarding for the Wayland display server protocols (newer Linux systems).
		- Session Logic:
			- *Integrated:* Similar to X11, it seamlessly integrates a remote application window into the local desktop environment.
- ***Stream-Based Protocols(Video Feeds)*** : GUI, technically function as *wrappers, transports, or video streams* rather than "native OS-level session protocols" (like RDP or SSH), or they are proprietary implementations that don't have a single standard name.
	- **WebRTC** : (used by Chrome Remote Desktop) Uses standard browser-based real-time communication protocols to stream the desktop as a high-definition video feed.
		- used by Chrome Remote Desktop
		- WebRTC (*Transport*) / Proprietary Google Stream (*Payload*).
		- This is the magic that allows "Clientless" access. You aren't using a new protocol, you are just tunneling an old one through a web page.
		- *Session Logic:* Shared (usually) or Authority (configurable).
	- NVIDIA **GameStream** : It is a *Video Streaming Protocol*, highly optimized for low latency, rather than a general-purpose desktop protocol.
		- Hardware-accelerated video streaming designed for gaming. It prioritizes low latency (*speed*) over perfect image retention.
		- used by Moonlight/Sunshine
		- This is unique because it prioritizes *Latency over Clarity*. RDP will freeze to keep the text sharp; GameStream will blur the image to keep the mouse moving smoothly.
		- *Session Logic:* Shared (Mirrors the physical monitor).
- ***Protocol Translators***
	- **HTML5 Gateway** : A server that translates RDP, VNC, or SSH traffic into WebSocket packets so a standard *web browser* can display the remote interface. It is not a protocol itself; it is a 'Translator' (Proxy).
		- used by Apache Guacamole
		- HTTPS / WebSockets (*Transport*) wrapping RDP/VNC (*Payload*).
		- Unlike TeamViewer or AnyDesk, the "Server" is a web server you host. You can visit `http://your-server/` from _any_ device (smart TV, locked-down library computer, phone) and get full access without installing a client app.
		- *Session Logic:* 'Inherited' (It behaves exactly like the protocol it is translating, RDP stays Authority, VNC stays Shared).

### Third Party Apps
Other than these basic protocols and methods, we also have third party apps that allow us to connect different machines. They have ways of handling the right protocols so you do not need to understand it, like:

|Solution Name|Windows|macOS|Linux|Chromebook|
|---|---|---|---|---|
|**TeamViewer**|✅|✅|✅|✅ (via Android app)|
|**AnyDesk**|✅|✅|✅|✅ (enable the "Accessibility Service" permission in ChromeOS settings for controlling a server)|
|**Splashtop**|✅|✅|✅|✅ (via Android app or web app)|
|**ConnectWise ScreenConnect**|✅|✅|✅|✅ (via Android app)|
|**Zoho Assist**|✅|✅|✅|✅ (via Android app or web app)|
|**Parsec**|✅|✅|✅|✔️|
|**VNC Connect**|✅|✅|✅|❌ (RealVNC provides a VNC Viewer Android app but I could not make it work)|
|**RustDesk**|✅|✅|✅|✅ (via Android app; requires Accessibility Service for control)|
|**Chrome Remote Desktop**|✅|✅|✅|✅ (Server support is for Attended "Remote Support" only; cannot be set up for Unattended access)|
|**Microsoft Remote Desktop**|✅|✔️|✔️|✔️ (via Android app)|
|**NoMachine**|✅|✅|✅|✔️ (via Android app or Linux container)|
|**Remmina**|❌|❌|✔️|✔️ (via Linux container)|

### Legend
- **Server**: The machine being controlled.
- **Client**: The machine controlling the server.
- ✅ : Can be installed as both Server (controlled) and Client (controlling).
- ✔️ : Can be installed as a Client only (can control).
- 👁️ : Only able to view (cannot control).
- ❌ : Cannot be installed as Client or Server (or is not supported natively).


### My Suggestion for best option for cross-OS
To use:
- **GUI**
	- Linux Mint as the server
		- VNC
			- Tight VNC - [[All Published Notes/Hardware/Computer/Operating System/LinuxMint/Remote Connections/Creating VNC server in LinuxMint - TightVNC\|Creating VNC server in LinuxMint - TightVNC]]
				- Creates a new user session, good for a machine with multiple users.
				- One user uses the server, other users can connect with VNC
			- x11vnc - [[All Published Notes/Hardware/Computer/Operating System/LinuxMint/Remote Connections/Creating VNC server in LinuxMint - x11vnc\|Creating VNC server in LinuxMint - x11vnc]]
				- allows us to share the session between server and client
				- This option can be considered a good choice if you only need the screen for a rush work. *Audio is not transferred together*.
- **CLI**
	- Chromebook as a server
		- SSH
			- [[All Published Notes/Hardware/Computer/Operating System/ChromeOS/Remote Connections/CLI/Creating SSH server in Chromebook\|Creating SSH server in Chromebook]] - highlighting Chromebook as it is not commonly seen in searches.
- **Third party solution**
	- RustDesk - [[All Published Notes/Hardware/Computer/Operating System/LinuxMint/Remote Connections/creating RustDesk server in LinuxMint\|creating RustDesk server in LinuxMint]]
		- best for LAN or same Wi-Fi connection.



---
