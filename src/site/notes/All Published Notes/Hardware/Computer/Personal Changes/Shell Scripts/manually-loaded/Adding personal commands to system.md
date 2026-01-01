---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/personal-changes/shell-scripts/manually-loaded/adding-personal-commands-to-system/"}
---

created: 2026-01-01
updated: 2026-01-01
### Adding personal commands to system
To use your `git_Tester2.sh` script like a standard system command (like `ls`, `cat`, `cd`), you need to place it in a directory that is included in your system's `PATH` environment variable. Your shell searches these directories for executable when you type a command.

Here's how you can do it:

##### Make the script executable
```sh
chmod +x git_Tester2.sh
```
##### Choose a suitable directory for your script
The best place for personal scripts is typically a `bin` directory within your home directory. This keeps your custom commands separate from system commands managed by your distribution.

- **Recommended for personal scripts:** `~/bin` (which expands to `/home/your_username/bin`)
- **For system-wide scripts (requires root/sudo):** `/usr/local/bin` (use this if you want all users on your system to be able to run it, but it's usually overkill for personal tools).

Let's assume you'll use `~/bin`.
##### Create the `~/bin` directory if it doesn't exist
using:
```sh
mkdir -p ~/bin
```
The `-p` flag ensures that the directory is created only if it doesn't already exist, and it won't throw an error if it does.
##### Move (or copy) your script to the chosen directory
It's usually best to move the script to its new permanent location if you do not want to retain your old copy. We rename this file while moving (`git_status_check`)

```sh
mv git_Tester2.sh ~/bin/git_status_check
```

##### Verify/Update your `PATH` environment variable
Many modern Linux distributions automatically add `~/bin` to your `PATH` when it exists. Let's check if yours does.

###### Check your current `PATH`
```sh
echo $PATH
```
###### If `~/bin` is NOT in your `PATH`
You need to add it to your shell's configuration file. For most users, use `echo $0` to find your shell language, then the shell configuration file will be `~/.shellrc` e.g., for `bash` it is `~/.bashrc`. 
Assuming it is `bash` and `~/.bashrc`.
- Open `~/.bashrc` in a text editor (e.g., `nano`, `gedit`, `code`): 
```sh
code ~/.bashrc
```
- Add the following lines to the very end of the file:
```sh
# Add personal bin directory to PATH if it exists
if [ -d "$HOME/bin" ]; then
    PATH="$HOME/bin:$PATH"
fi
```
- Save the file
- Exit the editor
- **Apply the changes:** 
	- To make the changes take effect in your current terminal session, run: `source ~/.bashrc`
	- Alternatively, you can simply close your current terminal and open a new one.

##### Test your script
Now, you should be able to run your script from any directory by just typing its name:
```sh
thegamelearner@thegamelearner-MS-7E12:~/Documents$ pwd
/home/thegamelearner/Documents
thegamelearner@thegamelearner-MS-7E12:~/Documents$ ls
 GithubNotes   git_Tester2.sh   git_Tester3.sh   git_Tester.sh  'Unity Projects'
thegamelearner@thegamelearner-MS-7E12:~/Documents$ git_status_check 

--- Up to Date Repositories (No Issues) ---
/home/thegamelearner/Documents/GithubNotes/TglBlog
  - This directory is up to date.

/home/thegamelearner/Documents/GithubNotes/hostedRepo
  - This directory is up to date.

--- Repositories with Issues (Sorted by Number of Types of Issues) ---

1 type(s) of issues:
/home/thegamelearner/Documents/Unity Projects/RPG_Practice
  - Untracked files found: 1


2 type(s) of issues:
/home/thegamelearner/Documents/GithubNotes/obsidianNotes
  - Untracked files found: 5
  - Changes not staged for commit: 4


3 type(s) of issues:
/home/thegamelearner/Documents/Unity Projects/2D-TopDown-Practice
  - Untracked files found: 13395
  - Changes not staged for commit: 12165
  - Ahead of remote by 1 commit(s).

thegamelearner@thegamelearner-MS-7E12:~/Documents$ 
```

#### Sorting PATH and in new line
use `echo $PATH | tr ':' '\n' | sort` to get output of `PATH` in individual lines and sorted alphabetically:
```sh hl:1,16
thegamelearner@thegamelearner-MS-7E12:~/Documents$ echo $PATH | tr ':' '\n' | sort
/bin
/home/thegamelearner/bin
/home/thegamelearner/bin
/home/thegamelearner/.dotnet/tools
/home/thegamelearner/.local/share/JetBrains/Toolbox/scripts
/home/thegamelearner/.local/share/JetBrains/Toolbox/scripts
/sbin
/snap/bin
/usr/bin
/usr/games
/usr/local/bin
/usr/local/games
/usr/local/sbin
/usr/sbin
thegamelearner@thegamelearner-MS-7E12:~/Documents$ echo $PATH | tr ':' '\n'
/home/thegamelearner/bin
/home/thegamelearner/bin
/usr/local/sbin
/usr/local/bin
/usr/sbin
/usr/bin
/sbin
/bin
/usr/games
/usr/local/games
/snap/bin
/home/thegamelearner/.dotnet/tools
/home/thegamelearner/.local/share/JetBrains/Toolbox/scripts
/home/thegamelearner/.local/share/JetBrains/Toolbox/scripts
thegamelearner@thegamelearner-MS-7E12:~/Documents$ 
```
