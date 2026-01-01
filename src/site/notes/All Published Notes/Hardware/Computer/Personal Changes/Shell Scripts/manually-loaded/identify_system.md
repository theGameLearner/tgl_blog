---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/personal-changes/shell-scripts/manually-loaded/identify-system/"}
---

created: 2026-01-01
updated: 2026-01-01
### identify_system
To identify the system, we need to know a few things like operating system.
Most Unix terminal based systems like Mac OS and Linux allow us to see this information, but figuring the right way to get it is hard.
I am making a command to give the necessary information to the end user.
This can be used to identify:
- Os type and name
	- Windows
	- Mac
	- Linux
		- Ubuntu/Debian
			- Linux Mint
		- Arch
			- ArchLinux, Garuda Linux
		- Red Hat
			- Fedora, CentOS
		- OpenSuse
- Desktop environment in use
	- gnome
	- kde
	- mate
	- cinnamon
	- xfce
	- jwm
- Architecture 
	- 64-bit
	- 32-bit
- Preferred app installer 
	- dpkg, apt, apt-get, aptitude, snapd, etc. (debian based)
	- pacman (Arch based)
	- rpm, yum (Red hat based)
	- zypper, (SUSE based)
- Shell
	- bash
	- zsh

### code
Usage:

| Command           | use case                |
| ----------------- | ----------------------- |
| `identify_system` | gives basic system info |

Code:
```sh title:identify_system.sh, hl:11-17,191-197
#!/bin/bash
# Auther : TheGameLearner

# Source the terminal colors script to make the functions available
source ~/.config/bash/terminal_colors.sh

# variables used by my program
TGL_OS_NAME=$(uname -s)

# Variables that will store the output
TGL_IS_OS_TYPE="" # Mac, Linux, Windows, or Unknown
TGL_IS_OS_LINUX_TYPE="" # tells the Linux type: debian, arch, rpm, suse
TGL_IS_OS_NAME="" # for storing the OS name like: Ubuntu, Linux Mint, Fedora, etc
TGL_IS_DE_NAME="" # for storing the desktop environment like: gnome, kde, mate, cinnamon, xfce, jwm
TGL_IS_ARCHITECTURE="" # for storing the architecture like: 64-bit or 32-bit
TGL_IS_COMMON_INSTALLERS=() # for storing the common installers like: apt, pacman, yum
TGL_IS_COMMON_SHELLS="" # for storing the current shell like: bash, zsh, etc


# --- Main OS Detection ---
case "$TGL_OS_NAME" in
    Darwin)
        # This is macOS
        echosuccess "Detected macOS."
        TGL_IS_OS_TYPE="Mac OS"
        TGL_IS_OS_NAME="macOS"
        ;;
    Linux)
        TGL_IS_OS_TYPE="Linux"
        # This is Linux. Now we need to be more specific.
        # Check for distribution-specific files from '/etc/os-release'
        if [ -f /etc/os-release ]; then
            . /etc/os-release
            # The ID variable contains the distribution name
            case "$ID" in
                ubuntu|debian|linuxmint)
                    echosuccess "Detected a Debian-based Linux distribution: ${NAME}."
                    TGL_IS_OS_LINUX_TYPE="debian"
                    TGL_IS_OS_NAME="${NAME}"
                    # Set installers for Debian-based systems
                    TGL_IS_COMMON_INSTALLERS=("apt" "apt-get" "aptitude" "dpkg")
                    # Check for snapd
                    if command -v snap >/dev/null; then
                        TGL_IS_COMMON_INSTALLERS+=("snap")
                    fi
                    ;;
                arch|garuda)
                    echosuccess "Detected an Arch-based Linux distribution: ${NAME}."
                    TGL_IS_OS_LINUX_TYPE="arch"
                    TGL_IS_OS_NAME="${NAME}"
                    # Set installers for Arch-based systems
                    TGL_IS_COMMON_INSTALLERS=("pacman")
                    # Check for yay or paru
                    if command -v yay >/dev/null; then
                        TGL_IS_COMMON_INSTALLERS+=("yay")
                    elif command -v paru >/dev/null; then
                        TGL_IS_COMMON_INSTALLERS+=("paru")
                    fi
                    ;;
                fedora|centos|rhel)
                    echosuccess "Detected an RPM-based Linux distribution: ${NAME}."
                    TGL_IS_OS_LINUX_TYPE="rpm"
                    TGL_IS_OS_NAME="${NAME}"
                    # Set installers for RPM-based systems
                    TGL_IS_COMMON_INSTALLERS=("dnf" "yum" "rpm")
                    ;;
                opensuse*)
                    echosuccess "Detected OpenSUSE Linux."
                    TGL_IS_OS_LINUX_TYPE="suse"
                    TGL_IS_OS_NAME="${NAME}"
                    # Set installers for OpenSUSE
                    TGL_IS_COMMON_INSTALLERS=("zypper")
                    ;;
                *)
                    echoerror "Detected an unknown Linux distribution: $ID."
                    TGL_IS_OS_LINUX_TYPE="Unknown"
                    TGL_IS_OS_NAME="Unknown_Linux_Distribution"
                    echohint "Check the linked document in /etc/os-release for more info."
                    ;;
            esac
        else
            echowarning "Detected a Linux system, but could not determine distribution, no file found at /etc/os-release"
            TGL_IS_OS_LINUX_TYPE="Unknown"
            TGL_IS_OS_NAME="Unknown_Linux_Distribution"
        fi

        # --- Desktop Environment Detection ---
        echoinfo "Detecting Desktop Environment..."
        
        # Method 1: Check the standard XDG_CURRENT_DESKTOP variable
        if [ -n "$XDG_CURRENT_DESKTOP" ]; then
            # Convert to lowercase for easier matching
            case "$(echo "$XDG_CURRENT_DESKTOP" | tr '[:upper:]' '[:lower:]')" in
                *gnome*)
                    TGL_IS_DE_NAME="gnome"
                    echosuccess "Detected Desktop Environment: GNOME"
                    ;;
                *kde*)
                    TGL_IS_DE_NAME="kde"
                    echosuccess "Detected Desktop Environment: KDE"
                    ;;
                *mate*)
                    TGL_IS_DE_NAME="mate"
                    echosuccess "Detected Desktop Environment: MATE"
                    ;;
                *cinnamon*)
                    TGL_IS_DE_NAME="cinnamon"
                    echosuccess "Detected Desktop Environment: Cinnamon"
                    ;;
                *xfce*)
                    TGL_IS_DE_NAME="xfce"
                    echosuccess "Detected Desktop Environment: XFCE"
                    ;;
                *jwm*)
                    TGL_IS_DE_NAME="jwm"
                    echosuccess "Detected Desktop Environment: JWM"
                    ;;
                *)
                    TGL_IS_DE_NAME="unknown"
                    echowarning "Could not determine Desktop Environment from XDG_CURRENT_DESKTOP: $XDG_CURRENT_DESKTOP"
                    ;;
            esac
        else
            # Method 2: Fallback to pgrep if XDG_CURRENT_DESKTOP is not set
            echoinfo "XDG_CURRENT_DESKTOP not set, falling back to pgrep..."
            if pgrep -x gnome-shell >/dev/null; then
                TGL_IS_DE_NAME="gnome"
                echosuccess "Detected Desktop Environment: GNOME"
            elif pgrep -x kwin_x11 >/dev/null || pgrep -x kwin_wayland >/dev/null; then
                TGL_IS_DE_NAME="kde"
                echosuccess "Detected Desktop Environment: KDE"
            elif pgrep -x mate-session >/dev/null; then
                TGL_IS_DE_NAME="mate"
                echosuccess "Detected Desktop Environment: MATE"
            elif pgrep -x cinnamon >/dev/null; then
                TGL_IS_DE_NAME="cinnamon"
                echosuccess "Detected Desktop Environment: Cinnamon"
            elif pgrep -x xfce4-session >/dev/null; then
                TGL_IS_DE_NAME="xfce"
                echosuccess "Detected Desktop Environment: XFCE"
            elif pgrep -x jwm >/dev/null; then
                TGL_IS_DE_NAME="jwm"
                echosuccess "Detected Desktop Environment: JWM"
            else
                TGL_IS_DE_NAME="unknown"
                echowarning "Could not determine Desktop Environment with pgrep."
            fi
        fi
        ;;
    CYGWIN*|MINGW*)
        echosuccess "Detected Windows via Cygwin/MSYS."
        TGL_IS_OS_TYPE="Windows"
        TGL_IS_OS_NAME="Windows"
        ;;
    *)
        # Handle other or unknown operating systems
        echowarning "Detected an unsupported OS: $TGL_OS_NAME. Setting default Name."
        TGL_IS_OS_TYPE="Unknown"
        TGL_IS_OS_LINUX_TYPE="Unknown"
        TGL_IS_OS_NAME="Unknown_OS"
        ;;
esac

# --- Architecture Detection ---
echoinfo "Detecting system architecture..."
ARCH=$(uname -m)
case "$ARCH" in
    x86_64|aarch64|arm64)
        TGL_IS_ARCHITECTURE="64-bit"
        echosuccess "Detected 64-bit architecture."
        ;;
    i386|i686)
        TGL_IS_ARCHITECTURE="32-bit"
        echowarning "Detected 32-bit architecture."
        ;;
    *)
        TGL_IS_ARCHITECTURE="Unknown"
        echoerror "Could not determine architecture."
        ;;
esac

# --- Current Shell Detection ---
echoinfo "Detecting current shell..."
TGL_IS_COMMON_SHELLS=$(basename "$SHELL")
echosuccess "Detected shell: $TGL_IS_COMMON_SHELLS"


# Print the final results
echo "---"
echoinfo "Final System Identification:"
echo "OS Type: ${TGL_IS_OS_TYPE}"
echo "OS Linux Type: ${TGL_IS_OS_LINUX_TYPE}"
echo "OS Name: ${TGL_IS_OS_NAME}"
echo "Desktop Environment: ${TGL_IS_DE_NAME}"
echo "Architecture: ${TGL_IS_ARCHITECTURE}"
echo "Common Installers: ${TGL_IS_COMMON_INSTALLERS[@]}"
echo "Current Shell: ${TGL_IS_COMMON_SHELLS}"
echo "---"

# Example of how you would use these variables in another script:
#
# if [ "$TGL_IS_OS_NAME" = "Linux Mint" ] && [ "$TGL_IS_DE_NAME" = "cinnamon" ]; then
#     echo "This is a Linux Mint system with Cinnamon."
# fi
```

adding this to bin:

```bash title:Usage, hl:1,4,5
thegamelearner@thegamelearner-MS-7E12:~$ cp ~/Documents/testSh/identify_system.sh $HOME/bin/identify_system 
thegamelearner@thegamelearner-MS-7E12:~$ which identify_system 
/home/thegamelearner/bin/identify_system
thegamelearner@thegamelearner-MS-7E12:~$ source ~/.bashrc
thegamelearner@thegamelearner-MS-7E12:~$ identify_system 
[SUCCESS] Detected a Debian-based Linux distribution: Linux Mint. 
[info] Detecting Desktop Environment...
[SUCCESS] Detected Desktop Environment: Cinnamon 
[info] Detecting system architecture...
[SUCCESS] Detected 64-bit architecture. 
[info] Detecting current shell...
[SUCCESS] Detected shell: bash 
---
[info] Final System Identification:
OS Type: Linux
OS Linux Type: debian
OS Name: Linux Mint
Desktop Environment: cinnamon
Architecture: 64-bit
Common Installers: apt apt-get aptitude dpkg
Current Shell: bash
---
thegamelearner@thegamelearner-MS-7E12:~$ 

```

---

[^1] [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/auto-loaded/bashrc Sub-Process/terminal_colors\|terminal_colors]]
[^2] [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/Adding personal commands to system\|Adding personal commands to system]]
[^3] '/etc/os-release' is a linker file that allows us to link to the actual file which stores OS related info

