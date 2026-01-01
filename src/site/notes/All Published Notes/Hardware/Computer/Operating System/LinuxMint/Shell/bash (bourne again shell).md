---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/operating-system/linux-mint/shell/bash-bourne-again-shell/"}
---

created: 2026-01-01
updated: 2026-01-01


The shell is the system terminal language which we use to talk to the hardware, the terminal is the application where you get an option to communicate with the system.
Verify you are using bash if you want to edit something specific to bash language:
```sh title:verify_shell.sh
$ echo $SHELL
```

To load personal changes or new commands that are exported, we can use the files in '/etc' directory for system wide configuration(needs administrator permission) or, we can use '~/' directory for user specific changes where `~` represents your HOME directory.

Types of shell:
- **Login Shell** - this is the terminal for non-UI based distributions, or the terminal you see when your system powers on. This is not used by casual users as everyone loves some UI, right?... RIGHT???
	- the login shell was what started your distribution behind your back when you logged in
- **non-login shells** - this is the terminal we are familiar with, whether you use the actual terminal your distribution came with, or something like *Tabby*, *alacritty* or *Kitty*
	- this shell is what i work on and hoping any mistakes are ignored.

File reading order:
- system switch on : `/etc/profile` → `/etc/bash.bashrc` → `BASH_ENV` -> `/etc/.bashrc`
- User Logs in : `~/.bash_profile` -> `~/.bash_login` -> `~/.profile` -> `~/.bashrc` -> `~/.bash_logout` (logout)
- System shut down : `/etc/.bash_logout`

> [!Warning]
> `.bash_profile` and `.profile` should not be used together


The diagram is here below. It's incomplete, but probably a reasonable place to start.
<img src="https://www.solipsys.co.uk/images/BashStartupFiles1.png" alt="https://shreevatsa.wordpress.com/wp-content/uploads/2008/03/bashstartupfiles1.png" style="max-width: 100%; height: auto;">

During login, we can use the following files to set up our machine:
- *.bash_profile*
	- The default profile option for bash language. 
	- It is loaded as soon as your profile is loaded, so when **system starts**
- *.bash_login*
	- If there is no `.bash_profile`, the system can load `.bash_login`. Most bash based distributions generally have `.bash_profile`, so this is rarely used now a days.
	- This is loaded when we **login** to a interactive or non-interactive shell
- *.profile*
	- all POSIX shell[^3] compatible shells will use this including bash, the restriction is that if either `.bash_profile` or `.bash_login` is available, bash will not read it.
	- Loaded when **system startup** calls a *login* shell
- *.bashrc*
	- This is the file which is used most for determining the flow of the code.
	- This is the file loaded when a **new interactive non-login shell is started**.
Similarly, during logout, we can define scripts to clean up the machine:
- *.bash_logout*
    - **This is read when you log out of a session** and is very good for cleaning things up when you leave (like resetting the Terminal Window Title)


---

[^1] : article by [shreevatsa](https://shreevatsa.wordpress.com/2008/03/30/zshbash-startup-files-loading-order-bashrc-zshrc-etc/ )
[^2] : article in [solipsys](https://www.solipsys.co.uk/new/BashInitialisationFiles.html) 
[^3]: The POSIX shell refers to the command-line shell that adheres to the Portable Operating System Interface (POSIX) standards. POSIX is a set of standards specified by the IEEE Computer Society to maintain compatibility among operating systems, particularly those based on Unix.

