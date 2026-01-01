---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/personal-changes/shell-scripts/auto-loaded/bashrc-sub-process/terminal-colors/"}
---

created: 2026-01-01
updated: 2026-01-01
### terminal_colors
We added colors to our '.bashrc' file, and created multiple `echo` alternatives with distinct styles:
The export ensures other processes from bin can also use them:

```sh title:~/.config/bash/terminal_colors.sh hl:21
#!/bin/bash

# Foreground Color Styles
ERROR_STYLE='\e[31m'
WARNING_STYLE='\e[33m'
SUCCESS_STYLE='\e[35m'
INFO_STYLE='\e[36m'
HINT_STYLE='\e[44m'
RESET='\e[0m'
# for more styles see 'Colors in terminal' in Obsidian notes


# Functions
echoerror() { echo -e "${ERROR_STYLE}[ERROR] $1${RESET}"; }
echowarning() { echo -e "${WARNING_STYLE}[WARNING] $1 ${RESET}"; }
echosuccess() { echo -e "${SUCCESS_STYLE}[SUCCESS] $1 ${RESET}"; }
echoinfo() { echo -e "${INFO_STYLE}[info] $1${RESET}"; }
echohint() { echo -e "${HINT_STYLE}$1${RESET}"; }

# Export the functions so they are available to all child processes 
export -f echoerror echowarning echosuccess echoinfo echohint

# Usage
# echoerror "File not found!"
# echowarning "Disk space low."
# echosuccess "Process completed."
# echoinfo "doing somethign"
# echohint "use 1st argument"
```

