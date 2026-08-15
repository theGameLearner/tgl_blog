---
{"dg-publish":true,"permalink":"/all-published-notes/computer-system/software/computer/personal-changes/shell-scripts/manually-loaded/git/unity-plugins/git-plugin-auto-status/"}
---

created: 2026-01-01
updated: 2026-08-01
### git_plugin_auto_status
Safely checks the status with folder renaming compatibility. Validates the repository, checks for uncommitted changes, temporarily renames directories to match remote structure, compares status and renames it back for working

- checks if we are running a git *directory*
- Gets the remote config
- renames the folders that have bad names
- git status in remote
- reverts the directory so we can work in case changes are needed
- pulls all remote tags

### Code
Usage:

| command                                      | Usage                                                     |
| -------------------------------------------- | --------------------------------------------------------- |
| `git_plugin_auto_status`                     | Default behavior, current directory is pulled from remote |
| `git_plugin_auto_status --path ~/my-project` | auto pulls the specified path repository                  |


Code:
```sh#!/bin/bash

# ==============================================================================
# Git Auto-Status Script
#
# Description:
#   Safely checks git status with folder renaming compatibility.
#   Temporarily renames directories to match remote structure, runs git status,
#   and restores original directory names automatically.
#
# Usage:
#   ./git_plugin_auto_status.sh [--path /path/to/repo]
#
# Arguments:
#   --path (optional): The file path to the Git repository.
#                      Defaults to the current directory (".").
#
# Exit Codes:
#   0: Success
#   1: Error (e.g., not a Git repo)
# ==============================================================================

# Exit immediately if a command exits with a non-zero status.
set -e

# --- Folder Name Configuration ---
# Define the names for local work vs. what's on the remote repository.
LOCAL_FOLDER_NAMES=("Samples" "Documentation")
REMOTE_FOLDER_NAMES=("Samples~" "Documentation~")

# Initialize variables with default values
TARGET_PATH="."

# --- Cleanup Function ---
# This function is called automatically when the script exits for any reason.
# It ensures that the renamed folders are always reverted to their original names.
function cleanup() {
    # Only run cleanup if we're in the correct directory
    if [[ ! -d "$TARGET_PATH" ]]; then
        return
    fi

    # Loop through the arrays to revert names from remote spec to local spec.
    for i in "${!REMOTE_FOLDER_NAMES[@]}"; do
        remote_name="${TARGET_PATH}/${REMOTE_FOLDER_NAMES[$i]}"
        local_name="${TARGET_PATH}/${LOCAL_FOLDER_NAMES[$i]}"

        # Check if the renamed folder exists before trying to move it back.
        if [[ -d "$remote_name" ]]; then
            mv "$remote_name" "$local_name"
        fi
    done
}

# Set the trap to ensure cleanup runs on script exit
trap cleanup EXIT

# --- Argument Parsing ---
while [[ $# -gt 0 ]]; do
    case "$1" in
        --path)
            if [[ -z "$2" ]]; then
                echo "Error: --path requires a directory argument." >&2
                exit 1
            fi
            TARGET_PATH="$2"
            shift 2
            ;;
        *)
            echo "Error: Unknown option '$1'" >&2
            echo "Usage: $0 [--path <directory>]" >&2
            exit 1
            ;;
    esac
done

# --- Initial Validation Checks ---

# 1. Check if the target directory actually exists.
if [[ ! -d "$TARGET_PATH" ]]; then
    echo "Error: Target directory '$TARGET_PATH' does not exist." >&2
    exit 1
fi

# 2. Check if the directory is a Git repository.
if ! git -C "$TARGET_PATH" rev-parse --is-inside-work-tree > /dev/null 2>&1; then
    echo "Warning: The directory '$TARGET_PATH' is not a Git repository." >&2
    exit 1
fi

# --- Folder Renaming (Pre-Status) ---
# We rename folders NOW so that git status sees the correct structure.
for i in "${!LOCAL_FOLDER_NAMES[@]}"; do
    local_name="${TARGET_PATH}/${LOCAL_FOLDER_NAMES[$i]}"
    remote_name="${TARGET_PATH}/${REMOTE_FOLDER_NAMES[$i]}"

    # Rename from the local name to the remote name to prepare for the check.
    if [[ -d "$local_name" ]]; then
        mv "$local_name" "$remote_name"
    fi
done

# --- Remote Repository Validation ---

# 1. Get the current branch name.
CURRENT_BRANCH=$(git -C "$TARGET_PATH" branch --show-current)
if [[ -z "$CURRENT_BRANCH" ]]; then
    echo "Error: Could not determine the current branch." >&2
    exit 1
fi

# 2. Find the upstream tracking branch
UPSTREAM_BRANCH=$(git -C "$TARGET_PATH" rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null)
if [[ -z "$UPSTREAM_BRANCH" ]]; then
    echo "Warning: The current branch '$CURRENT_BRANCH' is not tracking a remote branch." >&2
else
    # Parse the remote name from the upstream string
    REMOTE_NAME=$(echo "$UPSTREAM_BRANCH" | cut -d'/' -f1)

    # --- Fetch Incoming Changes ---
    echo "Fetching latest changes from remote '$REMOTE_NAME'..."
    git -C "$TARGET_PATH" fetch --tags --quiet "$REMOTE_NAME"
fi

# --- Git Status ---
echo "--- Git Status for '$TARGET_PATH' ---"
git -C "$TARGET_PATH" status

# The cleanup function (triggered by the trap) will automatically revert folder names
exit 0


# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #  # #  #
#   Usage                                                                     #
#-----------------------------------------------------------------------------#
#   # Default behavior (current directory)                                    #
#   ./git_plugin_auto_status.sh                                               #
#-----------------------------------------------------------------------------#
#   # With custom path                                                        #
#   ./git_plugin_auto_status.sh --path ~/my-project                           #
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

```


---

[^1] [[All Published Notes/Computer System/Software/Computer/Personal Changes/Shell Scripts/auto-loaded/bashrc Sub-Process/terminal_colors\|terminal_colors]]
[^2] [[All Published Notes/Computer System/Software/Computer/Personal Changes/Shell Scripts/manually-loaded/System/Adding personal commands to system\|Adding personal commands to system]]
[^3] [[All Published Notes/Computer System/Software/Computer/Personal Changes/Shell Scripts/manually-loaded/Git/Unity Plugins/git_plugin_auto_pull\|git_plugin_auto_pull]] - used as a reference.