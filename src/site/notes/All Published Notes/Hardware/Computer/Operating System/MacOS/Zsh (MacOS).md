---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/operating-system/mac-os/zsh-mac-os/"}
---

created: 2026-01-01
updated: 2026-01-01

Apple mainly uses z-shell, it is also read as `zsh`.
The shell is the system terminal language which we use to talk to the hardware, the terminal is the application where you get an option to communicate with the system.
Verify you are using zsh if you want to edit something specific to zsh language:
```sh title:verify_shell.sh
$ echo $SHELL
```


To load personal changes or new commands that are exported, we can use the files in '/etc' directory for system wide configuration(needs administrator permission) or, we can use '~/' directory for user specific changes where `~` represents your HOME directory.

The files go through the following order: `.zshenv` → `.zprofile` → `.zshrc` → `.zlogin` → `.zlogout`

During login, we can use the following files to set up our machine:
- *.zshenv* (environment variables) (optional)
    - the first file to be loaded, may create variables like your $PATH, $PAGER, or $EDITOR variables, so only to be used by advanced users
    - This can be considered as read and applied when you power on a machine
- *.zprofile* (login shell) (Preferred choice)
    - Modern approach, suggested place to set up the environment
    - Read and applied whenever a new user enters a session, for example when you choose a user profile to enter their creadentials.
    - gets loaded when you login or switch on the system, it is used to set up environment variable
    - Intended for setting environment variables and other shell-specific options that need to be available in login shells. 
    - Can be used to set up directories, source additional shell scripts, and add paths to $PATH 
    - Do not use this if you plan on using .zlogin, as both togeher may cause issues :: Conventionally, it's not recommended to use both .zlogin and .zprofile.
- *.zshrc* (interactive shell) (Recommended for shell related setup)
    - gets loaded when an insteractive shell(zsh based) is opened. like a new terminal window which we can interact with
    - Sourced for interactive shells. 
    - Used for setting up aliases, functions, shell options, and key bindings
    - Typically where you configure the look and feel of your terminal.
- *.zlogin* (login shell)
    - Old approach, generally not used if we are using .zprofile
    - intended for login shell configuration, similar to .zprofile. 
    - gets loaded after .zshrc
    - Conventionally, it's not recommended to use both .zlogin and .zprofile.
    - Can be used for displaying messages (like fortune, msgs) or creating files at login
    - Generally understood as the file read after a user is successfully logged in, or a successful switch user occurs

Similarly, during logout, we can define scripts to clean up the machine:
- *.zlogout*
    - **This is read when you log out of a session** and is very good for cleaning things up when you leave (like resetting the Terminal Window Title)






---

[^1]:
[^2]:

