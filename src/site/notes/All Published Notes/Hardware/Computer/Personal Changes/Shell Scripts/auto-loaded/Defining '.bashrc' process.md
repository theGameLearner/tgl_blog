---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/personal-changes/shell-scripts/auto-loaded/defining-bashrc-process/"}
---

created: 2026-01-01
updated: 2026-01-01
### Why?
I have the `.bashrc` file with lot of changes being made to my machine.
I would like to segregate the code to create specific files which does a specific action, and call them from `.bashrc` to make it easier to read and understand what my `.bashrc` does.

### Process
To make this as a standalone file, we will create a file in a location `.bashrc` can easily access:
```bash title:terminal hl:1,6-8,13
thegamelearner@thegamelearner-MS-7E12:~$ mkdir -p ~/.config/bash
thegamelearner@thegamelearner-MS-7E12:~$ ls -al ~/.config/bash/
total 8
drwxrwxr-x  2 thegamelearner thegamelearner 4096 Jul 19 17:08 .
drwxr-xr-x 42 thegamelearner thegamelearner 4096 Jul 19 17:08 ..
thegamelearner@thegamelearner-MS-7E12:~$ touch ~/.config/bash/terminal_colors.sh
thegamelearner@thegamelearner-MS-7E12:~$ chmod 600 ~/.config/bash/terminal_colors.sh
thegamelearner@thegamelearner-MS-7E12:~$ chmod +x ~/.config/bash/terminal_colors.sh
thegamelearner@thegamelearner-MS-7E12:~$ ls -al ~/.config/bash/
total 8
drwxrwxr-x  2 thegamelearner thegamelearner 4096 Jul 19 17:08 .
drwxr-xr-x 42 thegamelearner thegamelearner 4096 Jul 19 17:08 ..
-rwx--x--x  1 thegamelearner thegamelearner    0 Jul 19 17:08 terminal_colors.sh
thegamelearner@thegamelearner-MS-7E12:~$ code /home/thegamelearner/.config/bash/terminal_colors.sh 
thegamelearner@thegamelearner-MS-7E12:~$ # data saved from .bashrc to terminal_colors.sh
```

Update the code in `.bashrc`:

```sh title:.bashrc
# Load SSH agent setup from external script
if [ -f ~/.config/bash/terminal_colors.sh ]; then
    source ~/.config/bash/terminal_colors.sh
else
    echo "Warning: ~/.config/bash/terminal_colors.sh not found." >&2
fi
```

> [!Note]
> `source` (or `.`) runs the script in the **current shell**, preserving environment variables. Using `./script.sh` would spawn a subshell, and changes wouldn’t apply to your session.

Reload your `.bashrc` to verify:
```bash title:terminal
source ~/.bashrc
```

### Current changes:
We update the bashrc file, to use the following files:
- [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/auto-loaded/bashrc Sub-Process/terminal_colors\|terminal_colors]]
for any runtime script, we can refer to [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/Personal Commands\|Personal Commands]]

