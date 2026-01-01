---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/computer/personal-changes/shell-scripts/manually-loaded/gclone/"}
---

created: 2026-01-01
updated: 2026-01-01
### gclone
An intelligent Git clone wrapper that automatically converts Git URLs to their appropriate SSH format based on your SSH configuration. The script intelligently detects which SSH host/identity to use when you have multiple Git accounts configured in your `~/.ssh/config` file.
- **Automatic URL Conversion**: Converts HTTPS URLs to SSH format using your custom SSH hosts
- **SSH Config Integration**: Reads and respects your existing SSH configuration
- **Multi-Account Support**: Handles multiple Git accounts (personal, work, organization) seamlessly
- **Interactive Selection**: When multiple accounts exist for a domain, presents a user-friendly selection menu
- **Safe Dependency Checking**: Validates all requirements before execution

### Code
Usage:

| command                                                    | Usage                                                    |
| ---------------------------------------------------------- | -------------------------------------------------------- |
| `gclone https://github.com/tglGames-Plugins/repo.git`      | Converts HTTPS to SSH using appropriate host from config |
| `gclone git@github.com-tglGames:tglGames-Plugins/repo.git` | Uses existing SSH URL or converts based on config        |
| `gclone https://github.com/user/repo.git --branch=main`    | Passes additional git clone options                      |

in case we have multiple accounts and the script cannot identify which file to use:
```sh
[INFO] Multiple accounts found for github.com:
1) github.com-personal
2) github.com-work
3) Use default (github.com)
Selection [1-3]: 
```


Code:
```sh
#!/usr/bin/env bash

# ==============================================================================
# 1. Configuration & Metadata
# ==============================================================================
# This script requires Bash 4.0+ for the mapfile command
REQUIRED_BASH_MAJOR=4
REQUIRED_BASH_MINOR=0

# File paths - centralized for easy maintenance
COLOR_FILE="$HOME/.config/bash/terminal_colors.sh"
SSH_CONFIG="$HOME/.ssh/config"

# ==============================================================================
# 2. Dependency & Environment Validation
# ==============================================================================
check_dependencies() {
    echoinfo "Checking system dependencies..."
    
    # A. Check Bash Version
    if (( BASH_VERSINFO[0] < REQUIRED_BASH_MAJOR )) || \
       [[ ${BASH_VERSINFO[0]} -eq $REQUIRED_BASH_MAJOR && ${BASH_VERSINFO[1]} -lt $REQUIRED_BASH_MINOR ]]; then
        echoerror "This script requires Bash ${REQUIRED_BASH_MAJOR}.${REQUIRED_BASH_MINOR}+"
        echoerror "Your current version: ${BASH_VERSION}"
        echoerror "Please upgrade Bash: 'brew install bash' (macOS) or use package manager (Linux)"
        exit 1
    fi
    echoinfo "✓ Bash version $BASH_VERSION meets requirements"

    # B. Check for required binaries
    local reqs=("awk" "git" "ssh")
    echoinfo "Checking required tools: ${reqs[*]}"
    for tool in "${reqs[@]}"; do
        if command -v "$tool" >/dev/null 2>&1; then
            echoinfo "✓ Found: $tool ($($tool --version 2>&1 | head -n1))"
        else
            echoerror "Missing required tool: $tool"
            echoerror "Please install $tool using your package manager"
            exit 1
        fi
    done

    # C. Validate Config Files
    if [[ ! -f "$SSH_CONFIG" ]]; then
        echowarning "SSH config not found at: $SSH_CONFIG"
        echowarning "Custom SSH aliases will not be available"
        echowarning "Standard SSH format (git@domain:repo) will be used"
    else
        echoinfo "✓ SSH config found at: $SSH_CONFIG"
    fi

    # D. Color Setup & Function Fallbacks
    echoinfo "Setting up UI colors..."
    if [[ -f "$COLOR_FILE" ]]; then
        echoinfo "Loading custom colors from: $COLOR_FILE"
        # shellcheck source=/dev/null
        source "$COLOR_FILE"
        echoinfo "✓ Custom color file loaded"
    else
        echoinfo "No custom color file found, using system defaults"
    fi

    # Safe Fallbacks for UI Styles (use tput for terminal colors if available)
    RESET="${RESET:-$(tput sgr0 2>/dev/null || echo "")}"
    ERROR_STYLE="${ERROR_STYLE:-$(tput setaf 1 2>/dev/null || echo "")}"
    SUCCESS_STYLE="${SUCCESS_STYLE:-$(tput setaf 2 2>/dev/null || echo "")}"
    INFO_STYLE="${INFO_STYLE:-$(tput setaf 6 2>/dev/null || echo "")}"
    HINT_STYLE="${HINT_STYLE:-$(tput setaf 3 2>/dev/null || echo "")}"
    
    echoinfo "✓ Environment validation complete"
}

# ==============================================================================
# UI Helper Functions (all output to stderr to avoid polluting stdout)
# ==============================================================================
echoerror()   { echo -e "${ERROR_STYLE}[ERROR] $1${RESET}" >&2; }
echowarning() { echo -e "${HINT_STYLE}[WARNING] $1${RESET}" >&2; }
echosuccess() { echo -e "${SUCCESS_STYLE}[SUCCESS] $1${RESET}" >&2; }
echoinfo()    { echo -e "${INFO_STYLE}[INFO] $1${RESET}" >&2; }
echohint()    { echo -e "${HINT_STYLE}$1${RESET}" >&2; }

# ==============================================================================
# 3. Core Logic Functions
# ==============================================================================

# parse_url: Extract components from Git URL
# Input: Git URL (HTTPS, SSH, or SSH protocol)
# Output: "type|host/domain|path" or empty string if invalid
# Example: "https://github.com/user/repo.git" → "https|github.com|user/repo.git"
parse_url() {
    local url="$1"
    
    # Match HTTPS URLs (with optional port)
    if [[ "$url" =~ ^https?://([^/:]+)(:[0-9]+)?/(.+)$ ]]; then
        echo "https|${BASH_REMATCH[1]}|${BASH_REMATCH[3]}"
    # Match SSH URLs (git@host:path format)
    elif [[ "$url" =~ ^git@([^:]+):(.+)$ ]]; then
        echo "ssh|${BASH_REMATCH[1]}|${BASH_REMATCH[2]}"
    # Match SSH URLs with protocol (ssh://git@host/path format)
    elif [[ "$url" =~ ^ssh://git@([^/:]+)(:[0-9]+)?/(.+)$ ]]; then
        echo "ssh|${BASH_REMATCH[1]}|${BASH_REMATCH[3]}"
    fi
}

# find_hosts_by_hostname: Find all SSH Host entries that point to a specific domain
# Input: target domain (e.g., "github.com")
# Output: List of Host aliases (one per line)
find_hosts_by_hostname() {
    local target_domain="$1"
    [[ ! -f "$SSH_CONFIG" ]] && return
    
    echoinfo "Looking for SSH Host entries with HostName: $target_domain"
    awk -v target="$target_domain" '
        # Capture Host line: store all aliases (excluding wildcards)
        /^[[:space:]]*Host[[:space:]]+/ {
            delete aliases;
            for (i=2; i<=NF; i++) { 
                if ($i != "*") aliases[i-1] = $i 
            }
        }
        # When we find matching HostName, print all captured aliases
        /^[[:space:]]*HostName[[:space:]]+/ && $2 == target {
            for (a in aliases) print aliases[a]
        }
    ' "$SSH_CONFIG"
}

# is_ssh_alias: Check if a string exists as a Host entry in SSH config
# Input: host alias to check
# Output: Return 0 if found, 1 if not found
is_ssh_alias() {
    local host_to_check="$1"
    [[ ! -f "$SSH_CONFIG" ]] && return 1
    
    echoinfo "Checking if '$host_to_check' exists in SSH config..."
    awk -v host="$host_to_check" '
        # For each Host line, check if the alias matches exactly
        /^[[:space:]]*Host[[:space:]]+/ {
            for (i=2; i<=NF; i++) {
                if ($i == host) exit 0  # Found, exit with success
            }
        }
        END { exit 1 }  # Not found, exit with failure
    ' "$SSH_CONFIG" >/dev/null 2>&1
}

# ==============================================================================
# 4. Main Execution
# ==============================================================================
main() {
    echoinfo "=== Git Clone with SSH Host Selection ==="
    check_dependencies

    # Validate command line arguments
    if [[ $# -lt 1 ]]; then
        echoerror "No URL provided"
        echo ""
        echohint "Usage: gclone <git-url> [git-clone-options...]"
        echo ""
        echohint "Examples:"
        echohint "  gclone https://github.com/user/repo.git"
        echohint "  gclone git@github.com:user/repo.git"
        echohint "  gclone ssh://git@github.com/user/repo.git"
        echohint "  gclone https://github.com/user/repo.git --branch=main --depth=1"
        echo ""
        exit 1
    fi
    
    local url="$1"
    shift  # Remove URL, remaining arguments are for git clone
    
    echoinfo "Processing URL: $url"
    
    # Parse the URL into components
    local parsed=$(parse_url "$url")
    if [[ -z "$parsed" ]]; then
        echoerror "Invalid URL format: $url"
        echoerror "Supported formats:"
        echoerror "  - HTTPS: https://domain.com/user/repo.git"
        echoerror "  - SSH: git@domain.com:user/repo.git"
        echoerror "  - SSH with protocol: ssh://git@domain.com/user/repo.git"
        exit 1
    fi
    
    # Split parsed string into components
    IFS='|' read -r url_type identifier path <<< "$parsed"
    echoinfo "URL Type: $url_type, Identifier: $identifier, Path: $path"
    
    # Ensure repository path has .git extension
    if [[ "$path" != *.git ]]; then
        echoinfo "Adding .git extension to repository path"
        path="${path}.git"
    fi
    
    local final_host=""

    # Strategy 1: Check if identifier is already a valid SSH alias
    if [[ "$url_type" == "ssh" ]] && is_ssh_alias "$identifier"; then
        echoinfo "✓ Using existing SSH alias: $identifier"
        final_host="$identifier"
    else
        # Strategy 2: Treat identifier as domain and look for matching SSH hosts
        echoinfo "Looking for SSH hosts configured for domain: $identifier"
        
        # Read hosts into array (safe with mapfile in Bash 4.0+)
        mapfile -t hosts < <(find_hosts_by_hostname "$identifier")
        
        # Handle different cases based on number of hosts found
        case ${#hosts[@]} in
            0)
                echoinfo "No custom SSH hosts found for $identifier"
                echowarning "Using default: $identifier"
                final_host="$identifier"
                ;;
            1)
                echoinfo "Found 1 SSH host for $identifier: ${hosts[0]}"
                final_host="${hosts[0]}"
                ;;
            *)
                echoinfo "Found ${#hosts[@]} SSH hosts for $identifier"
                echoinfo "Please select which account to use:"
                
                # Build interactive selection prompt
                PS3=$(echo -e "${INFO_STYLE}Select account [1-$((${#hosts[@]} + 1))]: ${RESET}")
                
                # Present selection menu
                select choice in "${hosts[@]}" "Use default ($identifier)"; do
                    if [[ "$choice" == "Use default ($identifier)" ]]; then
                        echoinfo "Selected: Use default ($identifier)"
                        final_host="$identifier"
                        break
                    elif [[ -n "$choice" ]]; then
                        echoinfo "Selected: $choice"
                        final_host="$choice"
                        break
                    else
                        echoerror "Invalid selection. Please try again."
                    fi
                done
                ;;
        esac
    fi

    # Construct final SSH URL
    local final_url="git@${final_host}:${path}"
    
    # Log conversion details
    echo ""
    echoinfo "URL Conversion Summary:"
    echoinfo "  Original: $url"
    echoinfo "  Converted: $final_url"
    echo ""
    
    # Execute git clone with the converted URL
    echosuccess "Executing: git clone $final_url $*"
    echo ""
    
    if ! git clone "$final_url" "$@"; then
        echoerror "Git clone failed with exit code: $?"
        echoerror "Possible issues:"
        echoerror "  1. SSH key not loaded or incorrect"
        echoerror "  2. Repository URL is invalid"
        echoerror "  3. Network connectivity issue"
        echoerror "  4. SSH config has errors"
        exit 1
    fi
    
    echosuccess "Repository cloned successfully!"
}

# ==============================================================================
# Script Entry Point
# ==============================================================================
# Only run main if script is executed directly (not sourced)
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```


---


[^1] [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/auto-loaded/bashrc Sub-Process/terminal_colors\|terminal_colors]]
[^2] [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/Adding personal commands to system\|Adding personal commands to system]]
[^3] [[All Published Notes/Hardware/Computer/Personal Changes/Shell Scripts/manually-loaded/git_plugin_auto_pull\|git_plugin_auto_pull]] - used as a reference.