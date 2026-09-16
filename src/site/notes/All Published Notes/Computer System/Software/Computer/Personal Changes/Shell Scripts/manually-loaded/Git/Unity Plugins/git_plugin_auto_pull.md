---
{"dg-publish":true,"permalink":"/all-published-notes/computer-system/software/computer/personal-changes/shell-scripts/manually-loaded/git/unity-plugins/git-plugin-auto-pull/"}
---

created: 2026-01-01
updated: 2026-09-04
### git_plugin_auto_pull
Safely pulls remote changes with folder renaming compatibility. Validates the repository, checks for uncommitted changes, temporarily renames directories to match remote structure, pulls changes, and restores original directory names.

- checks if we are running a git *directory*
- Gets the remote config
- validates branch and pull in remote
- checks we have local changes to commit
- renames the folders that have bad names
- *pull* the remote changes and tags
### Code
Usage:

| command                                    | Usage                                                     |
| ------------------------------------------ | --------------------------------------------------------- |
| `git_plugin_auto_pull`                     | Default behavior, current directory is pulled from remote |
| `git_plugin_auto_pull --path ~/my-project` | auto pulls the specified path repository                  |


Code:
```sh
#!/bin/bash

# ==============================================================================
# Git Auto-Pull Script (Version 5)
#
# Description:
#   Safely pulls remote changes with folder renaming compatibility.
#   Validates the repository, checks for uncommitted changes, temporarily
#   renames directories to match remote structure, pulls changes, and restores
#   original directory names.
#
# Usage:
#   ./git_plugin_auto_pull.sh [--path /path/to/repo]
#
# Arguments:
#   --path (optional): The file path to the Git repository.
#                      Defaults to the current directory (".").
#
# Exit Codes:
#   0: Success (changes pulled)
#   1: Error (e.g., not a Git repo, local changes exist, pull failed)
#   2: No Action (already up-to-date)
# ==============================================================================

# Exit immediately if a command exits with a non-zero status.
set -e

# --- Load Terminal Colors or Fallback Functions ---
if [[ -f "$HOME/.config/bash/terminal_colors.sh" ]]; then
    source "$HOME/.config/bash/terminal_colors.sh" || true
fi

# Define fallback implementations if functions are not set
type echoerror >/dev/null 2>&1 || echoerror() { echo -e "$1"; }
type echowarning >/dev/null 2>&1 || echowarning() { echo -e "$1"; }
type echosuccess >/dev/null 2>&1 || echosuccess() { echo -e "$1"; }
type echoinfo >/dev/null 2>&1 || echoinfo() { echo -e "$1"; }
type echohint >/dev/null 2>&1 || echohint() { echo -e "$1"; }


# --- Folder Name Configuration ---
# Define the names for local work vs. what's on the remote repository.
LOCAL_FOLDER_NAMES=("Samples" "Documentation")
REMOTE_FOLDER_NAMES=("Samples~" "Documentation~")

# Initialize variables with default values
TARGET_PATH="."
abs_target_path=""

# --- Cleanup Function ---
# This function is called automatically when the script exits for any reason.
# It ensures that the renamed folders are always reverted to their original names.
function cleanup() {
    # Only run cleanup if TARGET_PATH exists
    if [[ ! -d "$TARGET_PATH" ]]; then
        return
    fi

    local display_path="${abs_target_path:-$TARGET_PATH}"

    echo "--- Running cleanup: Restoring folder names ---"
    # Loop through the arrays to revert names from remote spec to local spec.
    for i in "${!REMOTE_FOLDER_NAMES[@]}"; do
        remote_name="${TARGET_PATH}/${REMOTE_FOLDER_NAMES[$i]}"
        local_name="${TARGET_PATH}/${LOCAL_FOLDER_NAMES[$i]}"
        abs_remote_name="${display_path}/${REMOTE_FOLDER_NAMES[$i]}"
        abs_local_name="${display_path}/${LOCAL_FOLDER_NAMES[$i]}"

        # Check if the renamed folder exists before trying to move it back.
        if [[ -d "$remote_name" ]]; then
            if [[ -d "$local_name" ]]; then
                echo "------------------------------------------------------------" >&2
                echowarning "⚠️ MANUAL ACTION REQUIRED IN CLEANUP:" >&2
                echowarning "  Both '$abs_remote_name' and '$abs_local_name' exist." >&2
                echowarning "  The script will NOT automatically move or delete files." >&2
                echowarning "  Please inspect and resolve conflicts manually." >&2
                echo "------------------------------------------------------------" >&2
            else
                echo "Restoring '$abs_remote_name' back to '$abs_local_name'."
                mv "$remote_name" "$local_name"
            fi
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
                echoerror "Error: --path requires a directory argument." >&2
                exit 1
            fi
            TARGET_PATH="$2"
            shift 2
            ;;
        *)
            echoerror "Error: Unknown option '$1'" >&2
            echo "Usage: $0 [--path <directory>]" >&2
            exit 1
            ;;
    esac
done

# --- Initial Validation Checks ---
# 1. Check if the target directory actually exists.
if [[ ! -d "$TARGET_PATH" ]]; then
    echoerror "Error: Target directory '$TARGET_PATH' does not exist." >&2
    exit 1
fi

# Safely resolve absolute target path after validating directory exists
abs_target_path=$(cd "$TARGET_PATH" && pwd)

# 2. Check if the directory is a Git repository.
if ! git -C "$TARGET_PATH" rev-parse --is-inside-work-tree > /dev/null 2>&1; then
    echowarning "Warning: The directory '$TARGET_PATH' is not a Git repository." >&2
    exit 1
fi

echo "✅ Git repository validated at '$abs_target_path'."


# --- Remote Repository Validation ---
# 1. Get the current branch name.
CURRENT_BRANCH=$(git -C "$TARGET_PATH" branch --show-current)
if [[ -z "$CURRENT_BRANCH" ]]; then
    echoerror "Error: Could not determine the current branch. Are you in a detached HEAD state?" >&2
    exit 1
fi

# 2. Find the upstream tracking branch
UPSTREAM_BRANCH=$(git -C "$TARGET_PATH" rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null)
if [[ -z "$UPSTREAM_BRANCH" ]]; then
    echowarning "Warning: The current branch '$CURRENT_BRANCH' is not tracking a remote branch." >&2
    echo "Please set an upstream branch to pull from (e.g., git branch --set-upstream-to=origin/$CURRENT_BRANCH)." >&2
    exit 1
fi

# Parse the remote name and branch from the upstream string
REMOTE_NAME=$(echo "$UPSTREAM_BRANCH" | cut -d'/' -f1)
REMOTE_BRANCH=$(echo "$UPSTREAM_BRANCH" | cut -d'/' -f2-)

echo "✅ Current branch '$CURRENT_BRANCH' is tracking '$UPSTREAM_BRANCH'."

# --- Fetch and Check for Incoming Changes ---
echo "Fetching latest changes from remote '$REMOTE_NAME'..."
if ! git -C "$TARGET_PATH" fetch --tags --quiet "$REMOTE_NAME"; then
    echoerror "Error: Failed to fetch from remote repository '$REMOTE_NAME'." >&2
    exit 1
fi


# --- Folder Renaming (Pre-Pull) ---
# Rename folders BEFORE checking status so Git reads the expected remote structure.
echo "--- Temporarily aligning local folders with remote structure ---"
for i in "${!LOCAL_FOLDER_NAMES[@]}"; do
    local_name="${TARGET_PATH}/${LOCAL_FOLDER_NAMES[$i]}"
    remote_name="${TARGET_PATH}/${REMOTE_FOLDER_NAMES[$i]}"
    abs_local_name="${abs_target_path}/${LOCAL_FOLDER_NAMES[$i]}"
    abs_remote_name="${abs_target_path}/${REMOTE_FOLDER_NAMES[$i]}"

    # Rename from the local name to the remote name to prepare for the pull.
    if [[ -d "$local_name" ]]; then
        if [[ -d "$remote_name" ]]; then
            echo "------------------------------------------------------------" >&2
            echoerror "⛔ ERROR: Directory conflict detected!" >&2
            echoerror "  Both '$abs_local_name' and '$abs_remote_name' exist simultaneously." >&2
            echoerror "  Please inspect and resolve/remove '$abs_remote_name' manually." >&2
            echo "------------------------------------------------------------" >&2
            exit 1
        fi

        echo "Renaming '$abs_local_name' to '$abs_remote_name' for pull."
        mv "$local_name" "$remote_name"
    fi
done

# --- Check for Uncommitted Changes ---
# We shouldn't pull if there are uncommitted changes that could cause conflicts
# All untracked files are still ignored when pulling,
if [[ -n $(git -C "$TARGET_PATH" status --porcelain -uno) ]]; then
    echoerror "Error: You have uncommitted changes in your working directory." >&2
    echo "Please commit or stash your changes before pulling." >&2
    exit 1
fi

echo "✅ Working directory is clean."


# Check if the remote has commits that we don't have locally.
INCOMING_COMMITS=$(git -C "$TARGET_PATH" rev-list HEAD.."$UPSTREAM_BRANCH" --count)
if [[ "$INCOMING_COMMITS" -eq 0 ]]; then
    echo "✅ Your local branch is already up-to-date. No pull needed."
    exit 2
fi
echo "Found $INCOMING_COMMITS new commit(s) on '$UPSTREAM_BRANCH'."

# --- Pull Changes ---
echo "Pulling changes from '$UPSTREAM_BRANCH' (fast-forward only)..."
# Using --ff-only is safer for automation as it avoids creating unexpected merge commits.
if ! git -C "$TARGET_PATH" pull --ff-only --quiet "$REMOTE_NAME" "$REMOTE_BRANCH"; then
    echoerror "Error: Failed to pull changes from the remote repository." >&2
    echoerror "This might be due to merge conflicts or non-fast-forward changes." >&2
    echoerror "Please resolve manually and try again." >&2
    exit 1
fi

echosuccess "✅ Success! Changes have been pulled from the remote repository."

# The cleanup function (triggered by the trap) will automatically revert folder names
exit 0


# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #  # #  #
#   Usage                                                                     #
#-----------------------------------------------------------------------------#
#   # Default behavior (current directory)                                    #
#   ./git_plugin_auto_pull.sh                                                 #
#-----------------------------------------------------------------------------#
#   # With custom path                                                        #
#   ./git_plugin_auto_pull.sh --path ~/my-project                             #
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #


```


---


[^1] [[All Published Notes/Computer System/Software/Computer/Personal Changes/Shell Scripts/auto-loaded/bashrc Sub-Process/terminal_colors\|terminal_colors]]
[^2] [[All Published Notes/Computer System/Software/Computer/Personal Changes/Shell Scripts/manually-loaded/System/Adding personal commands to system\|Adding personal commands to system]]
[^3] [[All Published Notes/Computer System/Software/Computer/Personal Changes/Shell Scripts/manually-loaded/Git/Unity Plugins/git_plugin_auto_push\|git_plugin_auto_push]] - used as a reference.