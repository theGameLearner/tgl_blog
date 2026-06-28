---
{"dg-publish":true,"permalink":"/all-published-notes/software/computer/personal-changes/shell-scripts/shell-automation-scripts/download-and-install-git-packages/"}
---

created: 2026-06-27
updated: 2026-06-28

As you know, Linux has so many ways to install the packages that there is no way to make sure all packages are updated without getting the hands dirty. One of the source for apps in my machine is Github.

I want the option that I can get updated packages auto-installed in my machine whenever a new version is released in github, without having to manually look at updates.

I will use the `.deb` package from "https://github.com/kil0bit-kb/scrcpy-gui/releases" for this.

> [!Warning]
> This script is a template. The real script you use will have different dependencies, versions, tags, and other details. you can use this as a base to make your own
###### script
The script:
```sh
#!/bin/bash

# ==============================================================================
# HELP & SAFETY WARNINGS BLOCK
# ==============================================================================
# 1. PERMISSIONS: Must be executed with root/sudo privileges to install software.
# 2. COMPATIBILITY: Ensure the core CLI dependencies are pre-installed 
#    by extracting the tar file and placing the files in directories
# 3. SAFETY: Close any running instances of the app before running this update.
# 4. BACKGROUND AUTOMATION (Cron):
#    - If automated, ensure it doesn't conflict with system update schedules.
#    - Simultaneous use of 'apt' or the Mint Update Manager will cause failures.
# 5. LIMITS: Do not set to run excessively (e.g., inside .bashrc).
#    GitHub throttles unauthenticated API queries to 60 per hour per IP.
# ==============================================================================

# Exit immediately on error, unset variables, or pipeline failures
set -euo pipefail

# ==============================================================================
# USER CONFIGURATION BLOCK
# Change these variables to reuse this script for other GitHub .deb projects
# ==============================================================================
REPO="kil0bit-kb/scrcpy-gui"        # GitHub repository path (owner/repo)
PACKAGE_NAME="scrcpy-gui"          # Name used by local apt/dpkg system
FILE_EXTENSION=".deb"              # Target file extension to locate
# ==============================================================================

# 1. Ensure script is run with root/sudo privileges (required for installation)
if [ "$EUID" -ne 0 ]; then
    echo "Error: This script must be run with sudo or as root." >&2
    exit 1
fi

# 2. Check required tools (Replaced jq with python3, added pgrep)
for cmd in curl python3 wget dpkg sed cut pgrep; do
    if ! command -v "$cmd" &>/dev/null; then
        echo "Error: Required tool '$cmd' is missing. Please install it." >&2
        exit 1
    fi
done

# 3. Gatekeeper Check: Verify if another package manager has locked apt
echo "Verifying local package database status..."
if fuser /var/lib/apt/lists/lock >/dev/null 2>&1 || fuser /var/lib/dpkg/lock-frontend >/dev/null 2>&1; then
    echo "Error: The apt database is locked by another process (e.g., Update Manager)." >&2
    echo "Please wait for current updates to finish before running this script." >&2
    exit 1
fi

# 4. Gatekeeper Check: Verify core engine 'scrcpy' is present
if ! command -v scrcpy &>/dev/null; then
    echo "Error: Core binary 'scrcpy' engine is not installed on this system." >&2
    echo "Please install it first using: sudo apt install scrcpy adb" >&2
    exit 1
fi

# 5. Gatekeeper Check: Ensure no active app session is currently running
# Uses case-insensitive regex pattern check derived from the configuration name
if pgrep -f -i "$PACKAGE_NAME" >/dev/null 2>&1; then
    echo "Error: An active instance of '$PACKAGE_NAME' is currently running." >&2
    echo "Please save your work and close the application before updating." >&2
    exit 1
fi

# 6. Detect Native Host Architecture (e.g., amd64, arm64)
ARCH=$(dpkg --print-architecture)
echo "Checking for updates to $PACKAGE_NAME ($ARCH)..."

# 7. Safely Fetch GitHub API Data (with Rate Limit handling)
API_URL="https://api.github.com/repos/$REPO/releases"
TEMP_JSON="/tmp/github_releases_${PACKAGE_NAME}.json"

RESPONSE=$(curl -sL -w "%{http_code}" "$API_URL" -o "$TEMP_JSON" || true)

if [ "$RESPONSE" -ne 200 ]; then
    if grep -q "rate limit exceeded" "$TEMP_JSON" 2>/dev/null; then
        echo "Error: GitHub API rate limit exceeded. Try again later." >&2
    else
        echo "Error: Failed to fetch releases from GitHub (HTTP Status: $RESPONSE)." >&2
    fi
    rm -f "$TEMP_JSON"
    exit 1
fi

# 8. Parse JSON locally using Python 3 instead of jq
PARSED_DATA=$(python3 -c "
import json, sys

try:
    with open('$TEMP_JSON', 'r') as f:
        releases = json.load(f)
except Exception:
    print('ERROR:InvalidJSON')
    sys.exit(0)

valid_releases = []
for r in releases:
    if r.get('draft') or r.get('prerelease'):
        continue
    # Check if any asset matches extension and arch
    has_asset = any(a['name'].endswith('$FILE_EXTENSION') and '$ARCH' in a['name'] for a in r.get('assets', []))
    if has_asset:
        valid_releases.append(r)

if not valid_releases:
    print('ERROR:NoRelease')
    sys.exit(0)

# Sort by published date descending and take the newest stable release
valid_releases.sort(key=lambda x: x.get('published_at', ''), reverse=True)
latest = valid_releases[0]

tag = latest.get('tag_name', '')
url = ''
for a in latest.get('assets', []):
    if a['name'].endswith('$FILE_EXTENSION') and '$ARCH' in a['name']:
        url = a.get('browser_download_url', '')
        break

print(f'{tag}|||{url}')
" || true)

rm -f "$TEMP_JSON"

# Handle Python extraction errors elegantly
if [ -z "$PARSED_DATA" ] || [[ "$PARSED_DATA" == ERROR:* ]]; then
    echo "Error: Parsing failed or no valid stable release found matching '$ARCH' and '$FILE_EXTENSION'." >&2
    exit 1
fi

# Extract values from Python output string split by |||
LATEST_TAG=$(echo "$PARSED_DATA" | cut -d'|' -f1)
DOWNLOAD_URL=$(echo "$PARSED_DATA" | cut -d'|' -f4)

# 9. Extract and format version numbers
LATEST_VERSION=$(echo "$LATEST_TAG" | sed 's/^v//')
INSTALLED_VERSION=$(dpkg-query -W -f='${Version}' "$PACKAGE_NAME" 2>/dev/null | cut -d'-' -f1 || true)

if [ "$LATEST_VERSION" = "$INSTALLED_VERSION" ]; then
    echo "$PACKAGE_NAME is up to date (version $INSTALLED_VERSION)."
    exit 0
fi

echo "New version available: $LATEST_VERSION (installed: ${INSTALLED_VERSION:-none})"

# 10. Check if download URL exists
if [ -z "$DOWNLOAD_URL" ]; then
    echo "Error: Could not locate matching package asset in the release payload." >&2
    exit 1
fi

# 11. Download safely to a unique temporary file
DOWNLOADED_FILE=$(mktemp --suffix="$FILE_EXTENSION")
trap 'rm -f "$DOWNLOADED_FILE"' EXIT

echo "Downloading update from $DOWNLOAD_URL..."
# Checks if the environment is a terminal window to render visual progress bar
if [ -t 1 ]; then
    wget -q --show-progress -O "$DOWNLOADED_FILE" "$DOWNLOAD_URL"
else
    wget -q -O "$DOWNLOADED_FILE" "$DOWNLOAD_URL"
fi

# 12. Non-interactive installation block
echo "Installing $PACKAGE_NAME..."
export DEBIAN_FRONTEND=noninteractive

# Use modern apt path resolution first, fallback to dpkg if apt environment is constrained
if apt-get install -y "$DOWNLOADED_FILE" >/dev/null 2>&1; then
    echo "$PACKAGE_NAME successfully updated to $LATEST_VERSION!"
else
    dpkg -i "$DOWNLOADED_FILE" && apt-get install -f -y
    echo "$PACKAGE_NAME successfully updated to $LATEST_VERSION!"
fi
```

###### Testing
If the script runs successfully, you get logs like this:
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ sudo ~/bin/install_git_package_scrcpy_gui.sh 
Verifying local package database status...
Checking for updates to scrcpy-gui (amd64)...
New version available: 4.0.0 (installed: none)
Downloading update from https://github.com/kil0bit-kb/scrcpy-gui/releases/download/v4.0.0/ScrcpyGUI_4.0.0_amd64.deb...
/tmp/tmp.CKYsajVeuT.deb                                             100%[=================================================================================================================================================================>]   6.75M  7.87MB/s    in 0.9s    
Installing scrcpy-gui...
scrcpy-gui successfully updated to 4.0.0!
thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

if something fails, logs reflect the same:
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ ~/bin/install_git_package_scrcpy_gui.sh 
Error: This script must be run with sudo or as root.
thegamelearner@thegamelearner-MS-7E12 ~ $ sudo ~/bin/install_git_package_scrcpy_gui.sh 
[sudo] password for thegamelearner:           
Error: Required tool 'jq' is missing. Please install it.
thegamelearner@thegamelearner-MS-7E12 ~ $ sudo ~/bin/install_git_package_scrcpy_gui.sh 
[sudo] password for thegamelearner:           
Verifying local package database status...
Error: Core binary 'scrcpy' engine is not installed on this system.
Please install it first using: sudo apt install scrcpy adb
thegamelearner@thegamelearner-MS-7E12 ~ $ 
```

> [!Note]
> in case you do not have scrcpy, the logs will mention it, but not give you the package, installing it should not be part of an automated script that is supposed to install apps.

###### Automate the run

To automate running this script, you can add it to cronetab or you can add it as a necessary file for bashrc.
If using cronetab:
open the cronetab in the system's default editor
```sh
sudo crontab -e
```
you can add a line here to trigger running this script at fixed time or when some trigger condition is invoked, for example, every night at 12 O'Clock.
```
0 0 * * * /home/yourusername/update_scrcpy_gui.sh >/dev/null 2>&1
```

Make sure to save this file after line edit.


---

[^1]:
[^2]:

