---
{"dg-publish":true,"permalink":"/all-published-notes/computer-system/software/computer/personal-changes/shell-scripts/manually-loaded/git/unity-plugins/auto-login-to-git-in-terminal/"}
---

created: 2026-09-12
updated: 2026-09-12

I had issues in uploading the packages as I was not logged in properly, this can be a re-occurring issue, so I made this script to login to git on terminal for the registry that I want to publish in.

`git_plugin_auto_login`:
```bash
#!/usr/bin/env bash

set -e

# --- CONFIGURATION ---
REGISTRY="https://npm.pkg.github.com"
NPMRC_PATH="./.npmrc"

echo "=== Starting GitHub Packages NPM Verification ==="

# 1. Check if .npmrc exists in current working directory
if [[ ! -f "$NPMRC_PATH" ]]; then
  echo "ERROR: .npmrc file not found in current working directory ($(pwd))."
  exit 1
fi

echo "Found existing .npmrc in $(pwd)."

# 2. Verify login using existing .npmrc credentials
echo "Verifying authentication..."
if npm whoami --registry="${REGISTRY}" &>/dev/null; then
  LOGGED_USER=$(npm whoami --registry="${REGISTRY}")
  echo "SUCCESS: Logged in as '${LOGGED_USER}'."
else
  echo "WARNING: Could not verify login via 'npm whoami'. Please check your token inside .npmrc."
fi

```

This needs the `.npmrc` file to be in the same 'present working directory' in the following format:
`.npmrc`:
```npmrc
@tglGames-Plugins:registry=https://npm.pkg.github.com/
//npm.pkg.github.com/:_authToken=ghp_abc
registry=https://registry.npmjs.org/
```

where you can replace `ghp_abc` with the actual PAT you have from your github.

If you want to use a different path for `.npmrc`, update the `git_plugin_auto_login` script accordingly.




---

[^1]: 
[^2]: 

