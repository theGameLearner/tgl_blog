---
{"dg-publish":true,"permalink":"/all-published-notes/software/computer/personal-changes/shell-scripts/shell-automation-scripts/automate-commands-for-post-system-update/"}
---

created: 2026-06-28
updated: 2026-06-28

There will come a time when trying to automate things that we will want to automate updating apps that are not in packages or apt repository, like the package we download from git (e.g. [[All Published Notes/Software/Computer/Personal Changes/Shell Scripts/ShellAutomationScripts/Download and install git packages\|Download and install git packages]]). 

To automate, we can do either of these:
- Add the script to bashrc, this ensures it will run whenever new terminals are started or when system starts
- Add the script with a trigger logic to crontab as it can help automate the task based on time or other conditions
- Add this script to APT Hook File under `Post-Invoke` to be triggered even if the GUI is used to update the system.

As we have seen how to add commands to bashrc ([[All Published Notes/Software/Computer/Personal Changes/Shell Scripts/auto-loaded/Defining '.bashrc' process\|Defining '.bashrc' process]] and [[All Published Notes/Software/Computer/Personal Changes/Shell Scripts/manually-loaded/System/Adding personal commands to system\|Adding personal commands to system]]) and how to add commands in crontab ([[All Published Notes/Software/Computer/Personal Changes/Shell Scripts/ShellAutomationScripts/Download and install git packages#Automate the run\|Automate the command]]). We can now try to hook the process.


To automate by using apt hook, we want a wrapper script which will get triggered when we do anything with apt:
- This will be added to apt hook to run whenever any command uses apt
- This will also check if the conditions are right to run our custom script

##### Create the Wrapper
Let's create the script:
```bash
sudo fresh ~/bin/post_upgrade_wrapper.sh
```

Add a logic to the wrapper that ensures that it only runs the commands if we have done a system upgrade, and all logs will be stored in a file(`/var/log/scrcpy-gui-updater.log`) to be checked later:
```bash
#!/bin/bash

# ================================================================================================================
# CONFIGURATION
# ================================================================================================================
LOG_FILE="/var/log/scrcpy-gui-updater.log"

# ================================================================================================================
# 1. VISUAL FORMATTING BREAK
# Appends a clean separator layout block to make reading histories trivial
# ================================================================================================================
if [ -f "$LOG_FILE" ]; then
    echo -e "\n\n\n\n\n\n\n\n" >> "$LOG_FILE"
fi
echo "======================================================================" >> "$LOG_FILE"
echo "=== Upgrade Hook Triggered at $(date) ===" >> "$LOG_FILE"
echo "======================================================================" >> "$LOG_FILE"

# ================================================================================================================
# 2. RUNTIME ENVIRONMENT SANITY CHECK
# Ensure this only runs during true system management phases
# ================================================================================================================
if ! ps -ef | grep -E 'apt-get upgrade|apt upgrade|dist-upgrade|mintupdate|hello' | grep -v grep >/dev/null 2>&1; then
    echo "= Upgrade Hook skipped: Not an update command. =" >> "$LOG_FILE"
    exit 0
fi

# ================================================================================================================
# 3. BACKGROUND EXECUTION PIPELINE
# Fork the execution so APT finishes cleanly, then track the locks actively
# The main process exits and the subshell runs in the background, 
# The subshell checks every 0.5 seconds if the APT locks are released
# ================================================================================================================
(
    echo "Tracking system lock status in background subshell..." >> "$LOG_FILE"
    
    # Dynamic Tracking Loop: Watch lock files until they are released by the host process
    # This loop polls every 0.5 seconds, running as fast as possible without a single fixed delay
    while fuser /var/lib/apt/lists/lock >/dev/null 2>&1 || fuser /var/lib/dpkg/lock-frontend >/dev/null 2>&1; do
        sleep 0.5
    done
    
    echo "Package management locks completely cleared! Launching scripts..." >> "$LOG_FILE"

    # ------------------------------------------------------------------------------------------------------------
    # TARGET PROJECT AUTOMATIONS
    # Append any future standalone scripts directly to this block sequence!
    # ------------------------------------------------------------------------------------------------------------
    
    echo -e "\n[Target 1: Scrcpy GUI Update Engine]" >> "$LOG_FILE"
    /home/thegamelearner/bin/install_git_package_scrcpy_gui >> "$LOG_FILE" 2>&1

    # Settle down check: wait a moment if Target 1's installation just ran, to ensure apt is no longer locked/busy
    while fuser /var/lib/apt/lists/lock >/dev/null 2>&1 || fuser /var/lib/dpkg/lock-frontend >/dev/null 2>&1; do sleep 0.5; done

    echo -e "\n[Target 2: Zen Browser Deployment Engine]" >> "$LOG_FILE"
    # /home/thegamelearner/bin/install_git_package_zen_browser.sh >> "$LOG_FILE" 2>&1

) &
```

> [!Note]
> When the hooked files run, they run under root user because apt upgrade is a root process. So we cannot use `~` as a replacement for home folder path. `/home/thegamelearner` is my home (`~`) directory in the machine where I am running it.


> [!Note]
> Sometimes apt cannot create the log file even when running as root, so you may have to manually create the file before it can be used for logs.

Make this script executable:
```bash
sudo chmod +x ~/bin/post_upgrade_wrapper.sh
```

##### Create the Wrapper's hook
APT configuration snippets live inside `/etc/apt/apt.conf.d/`. Create a new configuration file there named `99-update-custom-packages`.

> [!Note]
> There is a very specific and functional reason for the number **`99`**. 
> The directory `/etc/apt/apt.conf.d/` acts as a modular repository of config snippets. When APT initializes, it sorts **all files inside alphabetically and numerically**, reading them sequentially from top to bottom. So by keeping a large number we are ensuring all default processes are completed before our file starts executing.

APT sorts these files **lexicographically** (alphabetically/character-by-character). In my machine, last used file was `90mintsystem` so 99 is a safe bet. You can use anything from 91 to 99.

Let's create the file which will hook to our file:
```bash
sudo fresh /etc/apt/apt.conf.d/99-update-custom-packages
```

Add line to run on post invoke of a DPkg package:
```bash
DPkg::Post-Invoke {"/home/thegamelearner/bin/install_git_package_template.sh";};
```
Ensure the permissions:
```bash
sudo chmod 644 /etc/apt/apt.conf.d/99-update-custom-packages
```


> [!Note]
> In `99-update-custom-packages` file as well, we are using full path instead of `~` as a replacement for home folder path (`/home/thegamelearner`).


To Test run `sudo apt update && sudo apt install --reinstall hello` which will update the app and try to run hook and stop as `upgrade` was not used.







---

[^1]:
[^2]:

