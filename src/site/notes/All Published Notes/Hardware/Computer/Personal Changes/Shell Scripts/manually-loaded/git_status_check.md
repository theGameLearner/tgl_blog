---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/personal-changes/shell-scripts/manually-loaded/git-status-check/"}
---

created: 2026-01-01
updated: 2026-01-01
### git_status_check
Using this command, we can recursively search all directories that are git repositories for their status and find out if we need to make any changes.
I made it using google Gemini, and the code is in bash.
How to use it now:

| Command                                      | use case                                                                                                          |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| ***`git_status_check`***                     | Default behavior (search current directory, show up-to-date)                                                      |
| `git_status_check /home/user/my_repos`       | Search a specific path (e.g., /home/user/my_repos), show up-to-date                                               |
| `git_status_check false`                     | Search current directory, skip up-to-date repositories                                                            |
| `git_status_check true`                      | Search current directory, explicitly show up-to-date repositories (same as `git_status_check`)                    |
| `git_status_check /home/user/my_repos false` | Search a specific path, skip up-to-date repositories                                                              |
| `git_status_check /home/user/my_repos true`  | Search a specific path, explicitly show up-to-date repositories, (same as `git_status_check /home/user/my_repos`) |

### code:

```sh title:git_status_check
#!/bin/bash

# --- Script Parameters ---
# Default values
SEARCH_PATH="."
SHOW_UP_TO_DATE="true" # Default to true

# Parse optional arguments
if [[ -n "$1" ]]; then
    LOWER_CASE_ARG1=$(echo "$1" | tr '[:upper:]' '[:lower:]')
    
    if [[ "$LOWER_CASE_ARG1" == "true" || "$LOWER_CASE_ARG1" == "false" ]]; then
        # First argument is a boolean flag, so use default path
        SHOW_UP_TO_DATE="$LOWER_CASE_ARG1"
        # If a second argument is provided, it should be the path
        if [[ -n "$2" ]]; then
            SEARCH_PATH="$2"
            # Optional: Add validation here if $2 is not a path, e.g., if it's another boolean.
            # For simplicity, we assume if the first argument is a flag, the second is a path.
        fi
    else
        # First argument is not a boolean, assume it's the path
        SEARCH_PATH="$1"
        # If a second argument is provided, it's the flag
        if [[ -n "$2" ]]; then
            LOWER_CASE_ARG2=$(echo "$2" | tr '[:upper:]' '[:lower:]')
            if [[ "$LOWER_CASE_ARG2" == "true" || "$LOWER_CASE_ARG2" == "false" ]]; then
                SHOW_UP_TO_DATE="$LOWER_CASE_ARG2"
            else
                echo "Warning: Invalid value for show_up_to_date flag. Expected 'true' or 'false'. Defaulting to '$SHOW_UP_TO_DATE'." >&2
            fi
        fi
    fi
fi

# Validate the provided path
if [[ ! -d "$SEARCH_PATH" ]]; then
    echo "Error: Provided path '$SEARCH_PATH' is not a valid directory." >&2
    exit 1
fi

# --- Spinner Functionality ---
spinner_chars=("-" "\\" "|" "/")
spinner_idx=0
spinner_pid="" # To store the PID of the background spinner process

start_spinner() {
  (
    # Run in a subshell to avoid interfering with main script's stdout
    while true; do
      echo -ne "\r${spinner_chars[spinner_idx]} Scanning repositories... " # \r returns cursor to start of line
      spinner_idx=$(( (spinner_idx + 1) % ${#spinner_chars[@]} ))
      sleep 0.1
    done
  ) &
  spinner_pid=$! # Store the PID of the background process
  # Disown the process so it doesn't become a zombie if parent exits abruptly
  disown "$spinner_pid" >/dev/null 2>&1 
}

stop_spinner() {
  if [[ -n "$spinner_pid" ]]; then
    kill "$spinner_pid" >/dev/null 2>&1 # Kill the background process
    echo -ne "\r\033[K" # Clear the line completely (\033[K is ANSI erase in line)
  fi
}

# Ensure spinner is stopped on script exit, even if there's an error or user interruption
trap stop_spinner EXIT

# --- Git Status Checking Function ---
# This function returns a formatted string containing all the information
# for a repository. Issue messages are joined by \x1E (Record Separator)
# to avoid issues with newlines during sorting/parsing.
# Format: "abs_dir|has_issues_flag|type_of_issues_count|total_individual_issues_count|message1\x1Emessage2\x1E..."
check_git_status() {
  local dir="$1"
  local has_issues=0 # Flag: 0 if clean, 1 if issues
  local abs_dir=$(realpath "$dir")
  local original_dir=$(pwd)
  declare -a issue_messages_array # Array to collect individual issue messages
  local type_of_issues_count=0 # Counter for distinct types of issues found
  local total_individual_issues_count=0 # Counter for total items/commits affected

  # Change to the directory
  cd "$dir" || {
    issue_messages_array+=("Error: Could not change to directory '$dir'")
    has_issues=1
    type_of_issues_count=1 # Directory access issue is one type
    total_individual_issues_count=1 # Counts as one individual issue (the access problem itself)
    # Ensure we return to original_dir before exiting the function for consistency
    cd "$original_dir" >/dev/null 2>&1
    # Return formatted string: Path|has_issues_flag|type_count|total_count|message (with \x1E)
    echo "$abs_dir|1|1|1|${issue_messages_array[0]}" 
    return # Exit the function
  }

  # Check for untracked files
  local untracked_files_count=$(git status --porcelain | grep "^\\?\\?" | wc -l)
  if [[ "$untracked_files_count" -gt 0 ]]; then
    issue_messages_array+=("  - Untracked files found: $untracked_files_count")
    has_issues=1
    type_of_issues_count=$((type_of_issues_count + 1))
    total_individual_issues_count=$((total_individual_issues_count + untracked_files_count))
  fi

  # Check for changes not staged for commit
  local not_staged_count=$(git status --porcelain | grep "^[MADRC ]" | wc -l)
  if [[ "$not_staged_count" -gt 0 ]]; then
    issue_messages_array+=("  - Changes not staged for commit: $not_staged_count")
    has_issues=1
    type_of_issues_count=$((type_of_issues_count + 1))
    total_individual_issues_count=$((total_individual_issues_count + not_staged_count))
  fi

  # Check for commits not pushed (ahead/behind remote)
  git remote update >/dev/null 2>&1 # Fetch updates silently

  local remote_tracking_branch=$(git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null)
  if [[ -n "$remote_tracking_branch" ]]; then
    local behind_or_ahead=$(git rev-list --left-right --count @{u}...HEAD 2>/dev/null)
    
    if [[ -n "$behind_or_ahead" ]]; then
      local parsed_values=($behind_or_ahead) # Split string by IFS (default space) into array
      local behind="${parsed_values[0]}"
      local ahead="${parsed_values[1]}"
      
      # Ensure values are numeric; default to 0 if not or empty.
      if ! [[ "$behind" =~ ^[0-9]+$ ]]; then behind=0; fi
      if ! [[ "$ahead" =~ ^[0-9]+$ ]]; then ahead=0; fi
      
      if [[ "$behind" -ne 0 ]]; then
        issue_messages_array+=("  - Behind remote by $behind commit(s).")
        has_issues=1
        type_of_issues_count=$((type_of_issues_count + 1))
        total_individual_issues_count=$((total_individual_issues_count + behind))
      fi
      if [[ "$ahead" -ne 0 ]]; then
        issue_messages_array+=("  - Ahead of remote by $ahead commit(s).")
        has_issues=1
        type_of_issues_count=$((type_of_issues_count + 1))
        total_individual_issues_count=$((total_individual_issues_count + ahead))
      fi
    fi
  fi

  # Check for merge conflicts (UU: unmerged paths in 'git status --porcelain')
  local conflict_files_count=$(git status --porcelain | grep "^UU" | wc -l)
  if [[ "$conflict_files_count" -gt 0 ]]; then
    issue_messages_array+=("  - Merge conflicts detected: $conflict_files_count")
    has_issues=1
    type_of_issues_count=$((type_of_issues_count + 1))
    total_individual_issues_count=$((total_individual_issues_count + conflict_files_count))
  fi
  
  # Return to the original directory
  cd "$original_dir" || {
    issue_messages_array+=("Error: Could not return to original directory '$original_dir'")
    has_issues=1 # Mark as issue if couldn't return
    type_of_issues_count=$((type_of_issues_count + 1)) # This is also a type of issue
    total_individual_issues_count=$((total_individual_issues_count + 1)) # Counts as one individual issue
  }

  # Format output: "abs_dir|has_issues_flag|type_of_issues_count|total_individual_issues_count|message1\x1Emessage2\x1E..."
  local status_flag="0" # 0 for clean, 1 for issues
  if [[ "$has_issues" -eq 1 ]]; then
      status_flag="1"
  fi
  
  local joined_messages=""
  if [[ ${#issue_messages_array[@]} -gt 0 ]]; then
    # Join messages with Record Separator (ASCII 0x1E). This ensures it's a single logical line for 'sort'.
    joined_messages=$(printf '%s\x1E' "${issue_messages_array[@]}")
    # Remove a single trailing record separator if present.
    joined_messages="${joined_messages%'\x1E'}"
  else
    # If no issues were found, provide the "up to date" message
    joined_messages="  - This directory is up to date."
  fi

  echo "$abs_dir|$status_flag|$type_of_issues_count|$total_individual_issues_count|$joined_messages"
}

# --- Main Script Execution ---
declare -a all_repo_results # Array to store formatted strings from check_git_status

start_spinner # Start the background spinner while scanning

# Find all .git directories and process them within the specified SEARCH_PATH
while IFS= read -r git_dir; do
  repo_dir="$(dirname "$git_dir")"
  repo_data=$(check_git_status "$repo_dir")
  all_repo_results+=("$repo_data")
done < <(find "$SEARCH_PATH" -name ".git" -type d)

stop_spinner # Stop the spinner once scanning is complete

# --- Post-processing and Printing ---
declare -a up_to_date_repos_output # Stores path and "up to date" message string
declare -a problematic_repos_for_sorting # Stores "type_of_issues_count|repo_path|full_data_string"

for entry in "${all_repo_results[@]}"; do
  # Split the entry based on the first three '|' delimiters to get core info
  IFS='|' read -r repo_path status_flag type_of_issues_count total_individual_issues_count raw_messages_string <<< "$entry"
  
  if [[ "$status_flag" -eq 0 ]]; then
    # For up-to-date repos, store the path and the "up to date" message
    up_to_date_repos_output+=("$repo_path\n$raw_messages_string")
  else
    # For problematic repos, store a string suitable for sorting: type_of_issues_count|path|original_full_entry
    problematic_repos_for_sorting+=("$type_of_issues_count|$repo_path|$entry")
  fi
done

# Sort problematic_repos_for_sorting numerically by the type of issues count (first field)
# `sort -t'|' -k1,1n` sorts by the first field as a number, using '|' as delimiter
IFS=

\n' sorted_problems=($(sort -t'|' -k1,1n <<<"${problematic_repos_for_sorting[*]}"))
unset IFS # Reset IFS to its default (space, tab, newline)

# --- Print Up-to-Date Repositories conditionally ---
if [[ "$SHOW_UP_TO_DATE" == "true" ]]; then
  if [[ ${#up_to_date_repos_output[@]} -gt 0 ]]; then
    echo -e "\n--- Up to Date Repositories (No Issues) ---"
    for entry in "${up_to_date_repos_output[@]}"; do
      echo -e "$entry\n" # Print path, "up to date" message, and a blank line
    done
  else
    echo -e "\nNo up to date repositories found."
  fi
fi

# --- Print Problematic Repositories (Sorted by Number of Types of Issues) ---
if [[ ${#sorted_problems[@]} -gt 0 ]]; then
  echo "--- Repositories with Issues (Sorted by Number of Types of Issues) ---"
  current_type_issue_count=-1 # To group by type of issues count
  for entry_for_sorting in "${sorted_problems[@]}"; do
    # Extract the type_of_issues_count and the original full data string
    # Store original IFS and set locally for this read command
    IFS_BAK="$IFS" 
    IFS='|' read -r type_issues_in_this_repo _ original_full_data <<< "$entry_for_sorting"
    IFS="$IFS_BAK" # Restore IFS
    
    # If the type of issues count changes, print a new heading
    if [[ "$type_issues_in_this_repo" -ne "$current_type_issue_count" ]]; then
      echo -e "\n$type_issues_in_this_repo type(s) of issues:"
      current_type_issue_count="$type_issues_in_this_repo"
    fi

    # Now parse the original full data string to get path and the raw messages string
    # Store original IFS and set locally for this read command
    IFS_BAK="$IFS"
    # Note: The first three underscores '_' skip repo_path, status_flag, type_of_issues_count.
    # The fourth underscore is for total_individual_issues_count.
    IFS='|' read -r repo_path _ _ _ raw_messages_string <<< "$original_full_data" 
    IFS="$IFS_BAK" # Restore IFS

    echo "$repo_path"
    
    # Split the raw_messages_string by Record Separator (\x1E) into an array for printing
    IFS_BAK="$IFS" 
    IFS=

\x1E' read -ra messages_array <<< "$raw_messages_string"
    IFS="$IFS_BAK" # Restore IFS

    # Print each message on a new line
    for msg in "${messages_array[@]}"; do
        echo "$msg"
    done
    echo # Add a blank line for readability
  done
else
  echo "No repositories with issues found."
fi

# --- Final Summary Message ---
# This message should only appear if no problematic repositories were found at all.
if [[ ${#problematic_repos_for_sorting[@]} -eq 0 ]]; then
  echo "All directories are up to date."
fi
```

