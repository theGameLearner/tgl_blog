---
{"dg-publish":true,"permalink":"/all-published-notes/software/computer/personal-changes/shell-scripts/shell-automation-scripts/install-git-package-zen-browser/"}
---

created: 2026-06-28
updated: 2026-06-28

Executable file for downloading and updating the Zen Browser:
```sh
#!/bin/bash

# ==============================================================================
# HELP & SAFETY WARNINGS BLOCK
# ==============================================================================
# 1. PERMISSIONS: Must be executed with root/sudo privileges.
# 2. SAFETY: Close any running instances of Zen Browser before updating.
# ==============================================================================

set -euo pipefail

REPO="zen-browser/desktop"          
PACKAGE_NAME="zen-browser"          
FILE_EXTENSION=".tar.xz"            # Updated for official Zen distribution

# 1. Ensure root privileges
if [ "$EUID" -ne 0 ]; then
    echo "Error: This script must be run with sudo or as root." >&2
    exit 1
fi

# 2. Check required system tools
for cmd in curl python3 wget dpkg sed cut pgrep tar; do
    if ! command -v "$cmd" &>/dev/null; then
        echo "Error: Required tool '$cmd' is missing." >&2
        exit 1
    fi
done

# 3. Gatekeeper Check: Verify if another package manager has locked apt
echo "Verifying local package database status..."
if fuser /var/lib/apt/lists/lock >/dev/null 2>&1 || fuser /var/lib/dpkg/lock-frontend >/dev/null 2>&1; then
    echo "Error: The apt database is locked by another process." >&2
    exit 1
fi

# 4. Gatekeeper Check: Ensure Zen Browser isn't actively running
if pgrep -x "zen" >/dev/null 2>&1 || pgrep -x "zen-bin" >/dev/null 2>&1; then
    echo "Error: An active instance of '$PACKAGE_NAME' is currently running." >&2
    exit 1
fi

# 5. Detect Architecture and map to Zen conventions
ARCH=$(dpkg --print-architecture)
if [ "$ARCH" = "amd64" ]; then
    ARCH="x86_64"
fi
echo "Checking for updates to $PACKAGE_NAME ($ARCH)..."

# 6. Fetch GitHub API Data
API_URL="https://api.github.com/repos/$REPO/releases/latest"
RESPONSE=$(curl -sSL -w "\n%{http_code}" "$API_URL")
HTTP_STATUS=$(echo "$RESPONSE" | tail -n1)
RELEASE_DATA=$(echo "$RESPONSE" | sed '$d')

if [ "$HTTP_STATUS" -ne 200 ]; then
    echo "Error: GitHub API responded with status code $HTTP_STATUS." >&2
    exit 1
fi

# 7. Parse Latest Available Version Tag
LATEST_VERSION=$(echo "$RELEASE_DATA" | python3 -c "
import sys, json
try:
    data = json.loads(sys.stdin.read())
    print(data.get('tag_name', '').strip())
except Exception:
    pass
")

if [ -z "$LATEST_VERSION" ]; then
    echo "Error: Failed to parse version metadata from GitHub." >&2
    exit 1
fi
LATEST_VERSION_CLEAN=$(echo "$LATEST_VERSION" | sed 's/^v//')

# 8. Fetch Current Installed Version Status from custom installation path
INSTALL_DIR="/opt/zen"
VERSION_FILE="$INSTALL_DIR/version.txt"
INSTALLED_VERSION=""
if [ -f "$VERSION_FILE" ]; then
    INSTALLED_VERSION=$(cat "$VERSION_FILE")
fi

if [ -n "$INSTALLED_VERSION" ]; then
    echo "Current installed version: $INSTALLED_VERSION"
    echo "Latest available version:  $LATEST_VERSION_CLEAN"
    if [ "$INSTALLED_VERSION" = "$LATEST_VERSION_CLEAN" ]; then
        echo "$PACKAGE_NAME is already up to date."
        exit 0
    fi
    echo "New version detected! Preparing upgrade sequence..."
else
    echo "$PACKAGE_NAME is not currently installed. Preparing clean deployment..."
fi

# 9. Extract Download URL matching tar.xz asset
DOWNLOAD_URL=$(echo "$RELEASE_DATA" | python3 -c "
import sys, json
try:
    data = json.loads(sys.stdin.read())
    assets = data.get('assets', [])
    for asset in assets:
        name = asset.get('name', '')
        url = asset.get('browser_download_url', '')
        if name.endswith('$FILE_EXTENSION') and 'linux' in name and '$ARCH' in name:
            print(url)
            break
except Exception:
    pass
")

if [ -z "$DOWNLOAD_URL" ]; then
    echo "Error: Could not locate a valid installation archive matching '$ARCH'." >&2
    exit 1
fi

# 10. Secure Temporal Environment & Download File
TEMP_DIR=$(mktemp -d)
trap 'rm -rf "$TEMP_DIR"' EXIT
DOWNLOAD_FILE="$TEMP_DIR/package$FILE_EXTENSION"

echo "Downloading archive..."
if ! wget -q --show-progress -O "$DOWNLOAD_FILE" "$DOWNLOAD_URL"; then
    echo "Error: Network download failed." >&2
    exit 1
fi

# 11. Extract and Deploy System Files
echo "Deploying application to $INSTALL_DIR..."
mkdir -p "$INSTALL_DIR"
# Purge old version assets while preserving user profiles
rm -rf "${INSTALL_DIR:?}"/*

# Extract tarball directly to /opt/zen, stripping the container root directory folder
tar -xf "$DOWNLOAD_FILE" -C "$INSTALL_DIR" --strip-components=1

# Save version stamp
echo "$LATEST_VERSION_CLEAN" > "$VERSION_FILE"

# 12. Wire up System Symlink Launcher
if [ ! -L /usr/local/bin/zen-browser ]; then
    ln -s "$INSTALL_DIR/zen" /usr/local/bin/zen-browser 2>/dev/null || true
fi

# 13. Generate Desktop Launcher Entry for Application Menu Integration
DESKTOP_ENTRY="/usr/share/applications/zen-browser.desktop"
if [ ! -f "$DESKTOP_ENTRY" ]; then
    echo "Generating system menu application launcher..."
    cat << 'EOF' > "$DESKTOP_ENTRY"
[Desktop Entry]
Version=1.0
Name=Zen Browser
Comment=Experience tranquility while browsing the web
GenericName=Web Browser
Exec=/usr/local/bin/zen-browser %u
Terminal=false
Type=Application
Icon=/opt/zen/browser/chrome/icons/default/default128.png
Categories=Network;WebBrowser;
MimeType=text/html;text/xml;application/xhtml+xml;application/xml;
EOF
    chmod 644 "$DESKTOP_ENTRY"
fi

echo "Success! $PACKAGE_NAME has been successfully synced to version $LATEST_VERSION_CLEAN."




```

Updates to Wrapper for ensuring this is also called:
```sh
#!/bin/bash

# ================================================================================================================
# CONFIGURATION
# ================================================================================================================
LOG_FILE="/var/log/scrcpy-gui-updater.log"

# ================================================================================================================
# 1. VISUAL FORMATTING BREAK
# Clear old data('not' Appends a clean separator layout block to make reading histories trivial)
# ================================================================================================================
if [ -f "$LOG_FILE" ]; then
    echo -e "\n\n\n\n\n\n\n\n" > "$LOG_FILE" # Clear old data and adds multiple new lines at the start of the logs
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


# ==============================================================================
# 3. USER TERMINAL NOTIFICATION
# This prints straight to your foreground console so you know it's working!
# ==============================================================================
echo ""
echo "▶ [Custom Update Engine] System upgrade hook caught. Moving tasks to background..."
echo "▶ [Custom Update Engine] To watch progress in real-time, run this command:"
echo "  ↳ tail -f $LOG_FILE"
echo ""

# ================================================================================================================
# 4. BACKGROUND EXECUTION PIPELINE
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
    /home/thegamelearner/bin/install_git_package_zen_browser >> "$LOG_FILE" 2>&1

) &




```


Finally, the hook in apt(does not change but adding as a reference):
```sh
thegamelearner@thegamelearner-MS-7E12 ~ $ sudo cat -n /etc/apt/apt.conf.d/99-update-custom-packages
     1  DPkg::Post-Invoke {"/home/thegamelearner/bin/post_upgrade_wrapper";};
     2
thegamelearner@thegamelearner-MS-7E12 ~ $ 
```


---

[^1]:
[^2]:

