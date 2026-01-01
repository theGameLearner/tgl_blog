---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/personal-changes/shell-scripts/manually-loaded/git-auto-push/"}
---

created: 2026-01-01
updated: 2026-01-01
### git_auto_push
A robust script that validates a Git repository, fetches updates, checks for large files, and then safely adds, commits, and pushes local changes. It is designed to prevent common issues like pushing to a repository that has new remote changes or accidentally committing large binary files.
- checks if we are running a git *directory*
- Gets the remote config
- validates branch and no pull in remote
- checks we have local changes to commit
- validates no large files > 50 MB
- *commit* the local changes
- push changes

### Code
Usage:

| command                                                     | Usage                                                                                  |
| ----------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `git_auto_push`                                             | Default behavior, current directory is pushed with auto-message with current date time |
| `git_auto_push --path ~/my-project --message "API updates"` | auto pushes the specified path with the specified message                              |
| `git_auto_push --path ../projects/alpha`                    | auto pushes the specified path with auto-message                                       |
| `git_auto_push --message "Hotfix for login issue"`          | auto pushes the current directory with the specified message                           |

code:
```sh

#!/bin/bash

# ==============================================================================================
# Git Auto-Push Script (Version 3)
#
# Description:
#   A robust script that validates a Git repository, fetches updates, checks
#   for large files, and then safely adds, commits, and pushes local changes.
#   It is designed to prevent common issues like pushing to a repository that
#   has new remote changes or accidentally committing large binary files.
#
# Usage:
#   ./git_auto_push.sh [--path /path/to/repo] [--message "Your commit message"]
#
# Arguments:
#   --path (optional): The file path to the Git repository.
#                      Defaults to the current directory (".").
#   --message (optional): The commit message to use.
#                         Defaults to a timestamp like "Auto-commit at 2025-08-07 10:26 PM".
#
# Exit Codes:
#   0: Success
#   1: Error (e.g., not a Git repo, remote has changes, large file found, push failed)
#   2: No Action (working directory was clean, no changes to commit)
# ==============================================================================================

# --- Configuration and Initial Setup ---

# Exit immediately if a command exits with a non-zero status.
set -e

# Initialize variables with default values
TARGET_PATH="."
COMMIT_MESSAGE=""

# --- Argument Parsing ---
# This loop processes command-line arguments like --path and --message.
while [[ $# -gt 0 ]]; do
    case "$1" in
        --path)
            if [[ -z "$2" ]]; then
                echo "Error: --path requires a directory argument." >&2
                exit 1
            fi
            TARGET_PATH="$2"
            shift 2 # Move past the flag and its value
            ;;
        --message)
            if [[ -z "$2" ]]; then
                echo "Error: --message requires a commit message string." >&2
                exit 1
            fi
            COMMIT_MESSAGE="$2"
            shift 2 # Move past the flag and its value
            ;;
        *)
            echo "Error: Unknown option '$1'" >&2
            echo "Usage: $0 [--path <directory>] [--message \"commit message\"]" >&2
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
# The `-C` flag tells Git to run the command *from* the TARGET_PATH
# without actually changing the script's current directory. This is safer.
if ! git -C "$TARGET_PATH" rev-parse --is-inside-work-tree > /dev/null 2>&1; then
    echo "Warning: The directory '$TARGET_PATH' is not a Git repository." >&2
    exit 1
fi

echo "✅ Git repository validated at '$TARGET_PATH'."


# --- Remote Repository Validation ---

# 1. Get the current branch name.
CURRENT_BRANCH=$(git -C "$TARGET_PATH" branch --show-current)
if [[ -z "$CURRENT_BRANCH" ]]; then
    echo "Error: Could not determine the current branch. Are you in a detached HEAD state?" >&2
    exit 1
fi

# 2. Find the upstream tracking branch (e.g., 'origin/main').
# This is a robust way to find the remote without hardcoding 'origin'.
UPSTREAM_BRANCH=$(git -C "$TARGET_PATH" rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null)
if [[ -z "$UPSTREAM_BRANCH" ]]; then
    echo "Warning: The current branch '$CURRENT_BRANCH' is not tracking a remote branch." >&2
    echo "Please set an upstream branch to push to (e.g., git push --set-upstream origin $CURRENT_BRANCH)." >&2
    exit 1
fi

# Parse the remote name and branch from the upstream string (e.g., "origin/main")
REMOTE_NAME=$(echo "$UPSTREAM_BRANCH" | cut -d'/' -f1)
REMOTE_BRANCH=$(echo "$UPSTREAM_BRANCH" | cut -d'/' -f2-)

echo "✅ Current branch '$CURRENT_BRANCH' is tracking '$UPSTREAM_BRANCH'."


# --- Fetch and Status Check ---

# 1. Fetch the latest changes from the remote.
# The `--quiet` flag suppresses unnecessary output.
echo "Fetching latest changes from remote '$REMOTE_NAME'..."
if ! git -C "$TARGET_PATH" fetch --quiet "$REMOTE_NAME"; then
    echo "Error: Failed to fetch from remote repository '$REMOTE_NAME'." >&2
    exit 1
fi

# 2. check for local commits not yet pushed. ###
# Check for commits that exist locally but not on the remote.
OUTGOING_COMMITS=$(git -C "$TARGET_PATH" rev-list "$UPSTREAM_BRANCH"..HEAD --count)
if [[ "$OUTGOING_COMMITS" -gt 0 ]]; then
    echo "Note: You have $OUTGOING_COMMITS local commit(s) that will be pushed."
fi

# 3. Check if the remote has commits that we don't have locally.
# If this count is greater than 0, we need to pull first.
INCOMING_COMMITS=$(git -C "$TARGET_PATH" rev-list HEAD.."$UPSTREAM_BRANCH" --count)
if [[ "$INCOMING_COMMITS" -gt 0 ]]; then
    echo "Error: Your local branch is behind the remote." >&2
    echo "There are $INCOMING_COMMITS new commit(s) on '$UPSTREAM_BRANCH'. Please 'git pull' first." >&2
    exit 1
fi

echo "✅ Your local branch is up-to-date. No pull needed."


# --- Pre-Commit Validations ---

# 1. Check if there are any changes to commit.
# `git status --porcelain` gives a clean, script-friendly output.
# If it's empty, there's nothing to do.
if [[ -z $(git -C "$TARGET_PATH" status --porcelain) ]]; then
    echo "No changes to commit. Working tree is clean."
    exit 2
fi

# 2. Check for large files BEFORE staging them.
MAX_FILE_SIZE_BYTES=52428800 # 50MB in bytes (50 * 1024 * 1024)
echo "Checking for files larger than 50MB..."

# Get a combined list of modified and untracked files.
# We set IFS (Internal Field Separator) to handle filenames with spaces correctly.
while IFS= read -r -d 

### Footnotes:
[^1] [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/auto-loaded/bashrc Sub-Process/terminal_colors\|terminal_colors]]
[^2] [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/Adding personal commands to system\|Adding personal commands to system]] \0' file; do
    full_path="$TARGET_PATH/$file"

    # Get file size in bytes. The `stat` command syntax differs between OSes.
    if [[ "$(uname)" == "Darwin" ]]; then
        # macOS syntax
        file_size=$(stat -f%z "$full_path")
    else
        # Linux syntax
        file_size=$(stat -c%s "$full_path")
    fi

    if [[ "$file_size" -gt "$MAX_FILE_SIZE_BYTES" ]]; then
        # Use awk for floating point division to get size in MB for the error message.
        size_mb=$(awk "BEGIN {printf \"%.2f\", $file_size/1024/1024}")
        
        echo "------------------------------------------------------------" >&2
        echo "⛔ Error: File size exceeds the 50MB limit." >&2
        echo "  File: $file" >&2
        echo "  Size: ${size_mb} MB" >&2
        echo "  Action: Please move this file out of the repository," >&2
        echo "          add it to .gitignore, or use Git LFS." >&2
        echo "------------------------------------------------------------" >&2
        exit 1 # This will exit the whole script
    fi
# `ls-files` is safer than parsing `status` for filenames that might contain spaces.
# Use process substitution `< <(...)` to feed the while loop. This avoids
# creating a subshell, so `exit 1` will terminate the entire script correctly.
# `git ls-files -z` and `read -d 

### Footnotes:
[^1] [[terminal_colors]]
[^2] [[Adding personal commands to system]] \0'` ensure filenames with spaces or
# special characters are handled correctly.
done < <(git -C "$TARGET_PATH" ls-files --modified --others --exclude-standard -z)

echo "✅ No large files found."


# --- Add, Commit, and Push ---

# 1. Set the commit message if a custom one wasn't provided.
if [[ -z "$COMMIT_MESSAGE" ]]; then
    # Note: The date format is "YYYY-MM-DD HH:MM AM/PM"
    COMMIT_MESSAGE="Auto-commit at $(date +'%Y-%m-%d %I:%M %p')"
fi

echo "Staging all changes..."
git -C "$TARGET_PATH" add .

echo "Committing with message: \"$COMMIT_MESSAGE\""
git -C "$TARGET_PATH" commit -m "$COMMIT_MESSAGE"

echo "Pushing changes to '$UPSTREAM_BRANCH'..."
# The final `git push` is the true test of write access.
if ! git -C "$TARGET_PATH" push "$REMOTE_NAME" "$CURRENT_BRANCH"; then
    echo "Error: Failed to push changes to the remote repository." >&2
    
    # --- FAILED PUSH RECOVERY ---
    # Attempt to undo the last commit to leave the repo in its original state.
    # This is safer than leaving a "dangling" commit.
    
    # First, check if this was the very first commit in the repo.
    # If it was, `HEAD~1` won't exist and `git reset` will fail.
    if git -C "$TARGET_PATH" rev-parse --verify HEAD~1 > /dev/null 2>&1; then
        echo "Attempting to reset the last commit locally..."
        git -C "$TARGET_PATH" reset --soft HEAD~1
        echo "Warning: The last commit was reset locally due to a failed push." >&2
    else
        echo "Warning: Cannot reset the initial commit of the repository." >&2
    fi
    exit 1
fi

echo "✅ Success! Changes have been pushed to the remote repository."
exit 0
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # 
#   Usage                                                             # 
#---------------------------------------------------------------------#
#   # Default behavior (current directory, auto-message)              #
#   ./git_auto_push.sh                                                #
#---------------------------------------------------------------------#
#   # With custom path and message                                    #
#   ./git_auto_push.sh --path ~/my-project --message "API updates"    #
#---------------------------------------------------------------------#
#   # With only path                                                  #
#   ./git_auto_push.sh --path ../projects/alpha                       #
#---------------------------------------------------------------------#
#   # With only message                                               #
#   ./git_auto_push.sh --message "Hotfix for login issue"             #
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # 


# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
#																	  #
#   # Updates needed                                                  #
#																	  #
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
#																	  #
#   > if we fail, log shows old commits due to failure				  #
#																	  #
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
#																	  #
#   > add custom echo messages when code is complete				  #
#																	  #
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

```

### Footnotes:
[^1] [[terminal_colors]]
[^2] [[Adding personal commands to system]] 