---
{"dg-publish":true,"permalink":"/all-published-notes/computer-system/software/computer/personal-changes/shell-scripts/manually-loaded/git/unity-plugins/git-plugin-auto-status/"}
---

created: 2026-01-01
updated: 2026-09-04
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

#!/bin/bash

# ==============================================================================
# Git Auto-Status Script (Version 5)
#
# Description:
#   Safely checks git status with folder renaming compatibility.
#   Temporarily renames directories to match remote structure, excludes meta
#   files associated with renamed folders/unstage patterns from status output,
#   runs git status, and restores original directory names automatically.
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

# --- Exclusion / Ignore Rules ---
# Pre-defined array of file patterns to exclude from status checks,
# matching the behavior of git_plugin_auto_push.
UNSTAGE_PATTERNS=(
    "*.meta"
)

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
    echo "Error: Target directory '$TARGET_PATH' does not exist." >&2
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

# --- Folder Renaming (Pre-Status) ---
# We rename folders NOW so that git status sees the correct structure.
echo "--- Temporarily aligning local folders with remote structure ---"
for i in "${!LOCAL_FOLDER_NAMES[@]}"; do
    local_name="${TARGET_PATH}/${LOCAL_FOLDER_NAMES[$i]}"
    remote_name="${TARGET_PATH}/${REMOTE_FOLDER_NAMES[$i]}"
    abs_local_name="${abs_target_path}/${LOCAL_FOLDER_NAMES[$i]}"
    abs_remote_name="${abs_target_path}/${REMOTE_FOLDER_NAMES[$i]}"

    # Rename from local name to remote name to prepare for git status
    if [[ -d "$local_name" ]]; then
        if [[ -d "$remote_name" ]]; then
            echo "------------------------------------------------------------" >&2
            echoerror "⛔ ERROR: Directory conflict detected!" >&2
            echoerror "  Both '$abs_local_name' and '$abs_remote_name' exist simultaneously." >&2
            echoerror "  Please inspect and resolve/remove '$abs_remote_name' manually." >&2
            echoerror "------------------------------------------------------------" >&2
            exit 1
        fi

        echo "Renaming '$abs_local_name' to '$abs_remote_name' for status check."
        mv "$local_name" "$remote_name"
    fi
done

# --- Remote Repository Validation ---

# 1. Get the current branch name.
CURRENT_BRANCH=$(git -C "$TARGET_PATH" branch --show-current)
if [[ -z "$CURRENT_BRANCH" ]]; then
    echoerror "Error: Could not determine the current branch. Are you in a detached HEAD state?" >&2
    exit 1
fi

# 2. Find the upstream tracking branch
UPSTREAM_BRANCH=$(git -C "$TARGET_PATH" rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null)
if [[ -n "$UPSTREAM_BRANCH" ]]; then
    # Parse the remote name from the upstream string
    REMOTE_NAME=$(echo "$UPSTREAM_BRANCH" | cut -d'/' -f1)

    # --- Fetch Incoming Changes ---
    echo "Fetching latest changes from remote '$REMOTE_NAME'..."
    git -C "$TARGET_PATH" fetch --tags --quiet "$REMOTE_NAME" || true
else
    echowarning "Warning: The current branch '$CURRENT_BRANCH' is not tracking a remote branch." >&2
fi

# --- Git Status with Exclusions ---
echo "--- Git Status for '$abs_target_path' ---"
# Construct status command array with targeted exclusions
STATUS_CMD=(git -C "$TARGET_PATH" status)

# Construct pathspec array to exclude targeted meta patterns from status output
EXCLUDE_PATHSPECS=()

# 1. Scope meta exclusions to the root folder meta files (e.g. :!/Samples.meta)
for local_folder in "${LOCAL_FOLDER_NAMES[@]}"; do
    EXCLUDE_PATHSPECS+=(":!/${local_folder}.meta")
done

# 2. Scope meta exclusions to files inside the renamed folders (e.g. :!Samples~/**/*.meta)
for remote_folder in "${REMOTE_FOLDER_NAMES[@]}"; do
    EXCLUDE_PATHSPECS+=(":!${remote_folder}/*.meta")
    EXCLUDE_PATHSPECS+=(":!${remote_folder}/**/*.meta")
done

# 3. Add custom non-meta patterns from UNSTAGE_PATTERNS
for pattern in "${UNSTAGE_PATTERNS[@]}"; do
    if [[ "$pattern" != "*.meta" ]]; then
        EXCLUDE_PATHSPECS+=(":!${pattern}")
    fi
done

# Run git status with pathspec exclusions passed after `--`
git -C "$TARGET_PATH" status -- "." "${EXCLUDE_PATHSPECS[@]}"

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