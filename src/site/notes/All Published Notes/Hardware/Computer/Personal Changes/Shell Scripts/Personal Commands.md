---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/personal-changes/shell-scripts/personal-commands/"}
---

created: 2026-01-01
updated: 2026-01-01
### Personal Commands
There are 2 ways to add your custom commands to your machine, one is to load commands in system through pre-configured directories that process all commands and make it available for all use cases (like /bin), another is to run commands from shell while we login or load a terminal to keep it available for the user.
#### Making System load Settings on start
These are changes we can do to the system to load and make our commands available when a system or terminal is loaded. Mainly we can use `.bashrc` or `.zshrc` for our use, but there are more than just these.
- terminal shell languages
	- bash
		- More Details: [[All Published Notes/Hardware/Computer/Operating System/LinuxMint/Shell/bash (bourne again shell)\|bash (bourne again shell)]] and [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/auto-loaded/Defining '.bashrc' process\|Defining '.bashrc' process]]
		- `.bash_profile` : This is the primary configuration file for a Bash login shell. When you log in (e.g., via SSH, at a console, or in a graphical terminal that starts a login shell), Bash first looks for and executes `~/.bash_profile`.
		- `.bash_login` : If `~/.bash_profile` is not found, Bash then looks for and executes `~/.bash_login`. This file is a fallback option.
		- `.profile` : If neither `~/.bash_profile` nor `~/.bash_login` is found, Bash then looks for and executes `~/.profile`. This is common for all POSIX shells (`sh`, `ksh`, `dash`, etc.)
		- `.bashrc` : runs before a new terminal instance is run
		- `.bash_logout` : this is run when you logout, can be used to clean up
	- zsh
		- More details: [[All Published Notes/Hardware/Computer/Operating System/MacOS/Zsh (MacOS)\|Zsh (MacOS)]]
		- *`.zshenv`* : system starts
		- *`.zprofile`* : Session changes or user logs in
		- *`.zshrc`* : when a interactive shell is loading (opening a new terminal window)
		- `.zlogin` : file runs when a new terminal is loaded and setup is complete (after `.zshrc`)
		- `.zlogout` : file runs when someone logs out of a session, mainly for cleanup

#### [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/Adding personal commands to system\|Adding personal commands to system]]
- System
	- `identify_system` : [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/identify_system\|identify_system]]
	- `list_extensions`: [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/list_extensions\|list_extensions]]
- Unity
	- `open_unity_project` : [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/open_unity_project\|open_unity_project]]
- Git
	- `gclone`: [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/gclone\|gclone]]
	- `git_auto_push` : [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/git_auto_push\|git_auto_push]]
	- `git_plugin_auto_pull` : [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/git_plugin_auto_pull\|git_plugin_auto_pull]]
	- `git_plugin_auto_push` : [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/git_plugin_auto_push\|git_plugin_auto_push]]
	- `git_plugin_auto_status`: [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/git_plugin_auto_status\|git_plugin_auto_status]]
	- `git_status_check`: [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/git_status_check\|git_status_check]]
- Unity plugins
	- 