---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/personal-changes/shell-scripts/manually-loaded/open-unity-project/"}
---

created: 2026-01-01
updated: 2026-01-01
### open_unity_project

Opens the current directory or a specified path with Unity.

| Command                             | use case                                                |
| ----------------------------------- | ------------------------------------------------------- |
| `open_unity_project`                | gives interactive options to select platform in 'pwd'   |
| `open_unity_project Android [path]` | opens the project at \[path\] with Android as platform  |
| `open_unity_project "" [path]`      | Open project with no platform change (uses last active) |

### Code:
```bash title:open_unity_project, hl:15,38,80,88-90
#!/bin/bash

# Get platform from first argument (optional)
PLATFORM=$1
# Get project path from second argument (optional)
PROJECT_PATH=${2:-$(pwd)}

# Verify project path is a Unity project
if [ ! -d "$PROJECT_PATH/Assets" ] || [ ! -d "$PROJECT_PATH/ProjectSettings" ]; then
    echo "Error: Not a valid Unity project (missing Assets or ProjectSettings directory)"
    exit 1
fi

# Find all installed Unity versions (Linux Mint path)
UNITY_VERSIONS=("$HOME/Unity/Hub/Editor"/*)

# Handle Unity version selection
if [ ${#UNITY_VERSIONS[@]} -eq 0 ]; then
    echo "No Unity versions found in ~/Unity/Hub/Editor/"
    exit 1
elif [ ${#UNITY_VERSIONS[@]} -eq 1 ]; then
    # If only one version exists, use it
    UNITY_VERSION=$(basename "${UNITY_VERSIONS[0]}")
    echo "Using the only installed Unity version: $UNITY_VERSION"
else
    # If multiple versions exist, list them
    echo "Available Unity versions:"
    for i in "${!UNITY_VERSIONS[@]}"; do
        echo "$((i+1)). $(basename "${UNITY_VERSIONS[$i]}")"
    done
    
    read -p "Enter the number of the Unity version to use: " VERSION_NUM
    if [[ ! "$VERSION_NUM" =~ ^[0-9]+$ ]] || [ "$VERSION_NUM" -lt 1 ] || [ "$VERSION_NUM" -gt ${#UNITY_VERSIONS[@]} ]; then
        echo "Invalid selection"
        exit 1
    fi
    
    UNITY_VERSION=$(basename "${UNITY_VERSIONS[$((VERSION_NUM-1))]}")
fi

# Handle platform selection if not provided
if [ -z "$PLATFORM" ]; then
    echo "Available platforms:"
    echo "1. StandaloneOSX"
    echo "2. StandaloneWindows"
    echo "3. StandaloneWindows64"
    echo "4. StandaloneLinux64"
    echo "5. iOS"
    echo "6. Android"
    echo "7. WebGL"
    echo "8. WSAPlayer"
    echo "9. PS4"
    echo "10. XboxOne"
    echo "11. tvOS"
    echo "12. Switch"
    
    read -p "Enter the number of the platform to use (or leave blank for default): " PLATFORM_NUM
    case $PLATFORM_NUM in
        1) PLATFORM="StandaloneOSX" ;;
        2) PLATFORM="StandaloneWindows" ;;
        3) PLATFORM="StandaloneWindows64" ;;
        4) PLATFORM="StandaloneLinux64" ;;
        5) PLATFORM="iOS" ;;
        6) PLATFORM="Android" ;;
        7) PLATFORM="WebGL" ;;
        8) PLATFORM="WSAPlayer" ;;
        9) PLATFORM="PS4" ;;
        10) PLATFORM="XboxOne" ;;
        11) PLATFORM="tvOS" ;;
        12) PLATFORM="Switch" ;;
        *) PLATFORM="" ;;  # No platform specified (Unity will use last active)
    esac
fi

echo "Opening project: $PROJECT_PATH"
echo "Using Unity version: $UNITY_VERSION"
[ -n "$PLATFORM" ] && echo "Active platform: $PLATFORM" || echo "Using last active platform."

# Set the correct Unity executable path for Linux
UNITY_EXECUTABLE="$HOME/Unity/Hub/Editor/$UNITY_VERSION/Editor/Unity"

if [ ! -f "$UNITY_EXECUTABLE" ]; then
    echo "Error: Unity executable not found at $UNITY_EXECUTABLE"
    exit 1
fi

# Build the command to launch Unity
UNITY_CMD="\"$UNITY_EXECUTABLE\" -projectPath \"$PROJECT_PATH\""
[ -n "$PLATFORM" ] && UNITY_CMD+=" -buildTarget $PLATFORM"
UNITY_CMD+=" -logFile \"$PROJECT_PATH/unity_log.txt\" > /dev/null 2>&1 &"

# Launch Unity
eval "$UNITY_CMD"
echo "Unity is launching in the background. Check 'unity_log.txt' for details."
```

added this to bin by moving the file.


### Footnotes
[^1] [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/Adding personal commands to system\|Adding personal commands to system]]

