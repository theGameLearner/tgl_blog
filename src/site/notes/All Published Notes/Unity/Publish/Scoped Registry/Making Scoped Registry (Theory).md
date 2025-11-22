---
{"dg-publish":true,"permalink":"/all-published-notes/unity/publish/scoped-registry/making-scoped-registry-theory/"}
---

created: 2025-10-04
updated: 2025-11-22

> [!Note]
> This file is generated from Gemini AI, I might do slight updates or modification if I feel any information needs editing.
# Comprehensive Workflow for Publishing Unity Packages to Public GitHub Packages NPM Registry

This report establishes a prescriptive, phased workflow for the `tglGames-Plugins` organization to migrate its existing Unity packages, currently distributed via GitHub Git URLs, into a managed, version-controlled repository utilizing the Public GitHub Packages NPM registry. The process covers package re-scoping, authentication, publication, dependency refactoring, and defines the subsequent transition to a secured Private registry model.

---

## PHASE I: Preparation and Package Re-Scoping for GitHub Packages

The foundational step involves modifying package metadata to satisfy the specific scoping requirements enforced by both the Unity Package Manager (UPM) and the GitHub Packages NPM registry.

### 1. Architectural Overview: GitHub Packages and UPM Integration

Successful integration hinges on correctly routing package requests and adhering to dual naming conventions.

#### The Role of Scoped Registries in Unity Package Manager (UPM)

Unity Package Manager utilizes the `scopedRegistries` definition within the project's `Packages/manifest.json` file to direct package resolution requests.1 This mechanism is necessary because UPM must be explicitly instructed to search a custom registry, such as GitHub Packages, instead of defaulting to the official Unity registry or the standard NPM registry.2 By defining a scope prefix, UPM effectively isolates lookups for proprietary packages, ensuring efficiency and preventing unauthorized data leakage.

#### Mapping Unity Reverse-Domain Names to NPM Scopes

GitHub Packages strictly supports only scoped NPM packages, which are formatted as `@NAMESPACE/PACKAGE-NAME`.3 Consequently, the existing Unity package naming convention, which uses reverse-domain notation (`com.tglgames.packagename`), must be adapted to incorporate the organization's GitHub scope, `@tglGames-Plugins`.

The package name structure must satisfy both requirements simultaneously: the NPM scope (`@tglGames-Plugins`) is prepended to the Unity-specific reverse-domain name (`com.tglgames.packagename`). This results in a final, fully qualified NPM package name, such as `"name": "@tglGames-Plugins/com.tglgames.corepackage"`.

This convention is mandatory for registry resolution. If the UPM `scopedRegistries` configuration later specifies that all packages starting with `com.tglgames` should be routed to the custom GitHub registry, this routing mechanism successfully isolates package lookups. The advantage of this specific routing is that it ensures all other package requests, particularly those for standard Unity packages (`com.unity.*`) or non-scoped external dependencies, continue to be fetched from their appropriate default NPM registries without incurring potential latency or authentication complexities associated with the custom registry.4

### 2. Pre-Publication Configuration Checklist

Prior to attempting publication, specific authentication and metadata preparation steps must be completed.

#### Generating the Classic Personal Access Token (PAT) for Publishing

Authentication for publishing to GitHub Packages requires a Personal Access Token (PAT).5 GitHub generally recommends the use of Personal Access Tokens (classic) for traditional NPM publishing operations, as fine-grained tokens may have limitations in this context.5

The PAT must be granted the appropriate permissions for this task. The absolute minimum required scope is `write:packages`, which permits the token owner to upload and publish packages to GitHub Packages.5 It is also highly recommended to include the

`repo` scope if the package is linked to a specific GitHub repository, which is standard practice for Unity packages.3 It is critical to note that even with the correct PAT scopes, the user generating the token must possess the necessary organizational write permissions to publish packages to the designated

`tglGames-Plugins` package space.5

This PAT, used for publishing, must be managed carefully. Adhering to the principle of least privilege dictates that this publishing token (which holds `write:packages` permission) must be distinct from any read-only tokens used later by developers solely for consuming packages.

#### Setting Up the Local NPM Authentication Context (`.npmrc`)

The local NPM environment must be configured to correctly route publishing commands for the specific scope to GitHub Packages. This is achieved by creating or updating a global or project-specific `.npmrc` file with a scope definition.

The required definition ensures that the NPM CLI directs publish operations for the organization's scope to the GitHub endpoint:

Ini, TOML

```
@tglGames-Plugins:registry=https://npm.pkg.github.com/
```

This configuration line establishes the necessary context for the NPM CLI, directing subsequent `npm publish` commands to the correct registry URL associated with the organization’s scope.3

#### Package Metadata Audit and Refactoring (`package.json`)

The `package.json` file within each existing Unity package repository must be modified systematically.

1. **Name Field Update:** The existing reverse-domain name (e.g., `com.tglgames.core`) must be replaced with the new, scoped NPM name (e.g., `@tglGames-Plugins/com.tglgames.core`). The package names and scopes must utilize only lowercase letters.3
    
2. **Version Field Verification:** A valid Semantic Versioning string (e.g., `1.0.0`) must be confirmed or assigned, as this is the version tag that will be published and used for dependency resolution.
    
3. **Repository Linking:** To improve package discoverability, leverage security features like Dependabot alerts, and automatically grant GitHub Actions workflow access, the `repository` field should be included in `package.json`, pointing to the source GitHub repository.3 Although the GitHub NPM registry supports granular package permissions separate from repository permissions 7, linking the repository is metadata best practice.
    

---

## PHASE II: Public Deployment Workflow

This phase details the authentication steps and the critical commands required to execute the public publication, followed by the essential refactoring of internal dependencies.

### 3. Authentication and Login to GitHub Packages NPM

The local NPM environment must be logged into the GitHub Packages registry using the generated PAT.

#### Command Line Login Procedure using PAT

GitHub requires a specific authentication flow when using the command line with PATs. The recommended procedure avoids the default browser-based login, ensuring the PAT is securely used.3

The command must specify the scope and the registry URL:

Bash

```
npm login --scope=@tglGames-Plugins --auth-type=legacy --registry=https://npm.pkg.github.com
```

When prompted, the developer should enter their GitHub username and the previously generated Classic PAT (with `write:packages` scope) as the password.3 This procedure stores the PAT securely in the user's NPM configuration, specifically scoped to the

`tglGames-Plugins` registry endpoint, preventing the token from being used against the standard global NPM registry.

### 4. Executing the Public Publication

Once authenticated, packages can be published. It is essential to explicitly override the default visibility setting.

#### The $`npm publish --access public` Command Execution

When publishing scoped packages, the default visibility setting is **private**.3 To meet the requirement of initial public visibility, the developer must explicitly pass the

`--access public` flag during publication.

For each package repository, navigate to the root directory containing the updated `package.json` and execute:

Bash

```
npm publish --access public
```

If this flag were omitted, the package would default to private, requiring an immediate administrative step via the GitHub UI or API to adjust its visibility, disrupting the planned public release workflow.

#### Verification of Package Visibility on GitHub

Following a successful publication, the developer must verify the package listing on the `tglGames-Plugins` organization’s GitHub Packages page. Confirmation is required that the package status is marked as Public and that all associated metadata (version number, description, and linked repository) is accurate.5

### 5. Refactoring Internal Package Dependencies

This step transforms the organization's package ecosystem from a Git-tracking model to a managed semantic versioning system. This refactoring must occur _after_ all packages have been successfully published.

#### Strategy for Replacing Git URLs with Published Version Numbers

The existing dependency resolution relies on Git URLs, which require explicit repository paths and potentially branch/tag specifications (e.g., `git+https://github.com/tglgames-Plugins/core.git#1.0.0`).9 After publication, these references must be replaced with the new scoped NPM name and a semantic version number.

For example, a dependency block within `package.json` would transition from:

JSON

```
{
  "dependencies": {
    "com.tglgames.utility": "git+https://github.com/tglgames-Plugins/utility.git#1.0.0"
  }
}
```

To the versioned, scoped reference:

JSON

```
{
  "dependencies": {
    "@tglGames-Plugins/com.tglgames.utility": "1.0.0"
  }
}
```

The success of this refactoring depends entirely on the prior step: all dependent packages must first be published to the registry using their initial version (e.g., `1.0.0`). If developers attempt to update dependency files before the target packages are resolvable via the registry, dependency management tools (like NPM or UPM) will fail immediately when attempting to fetch or install the necessary components. This confirms the necessity of a strict, iterative process: publish all packages, then synchronize dependency files across all projects.

For a large number of packages, dependency updates can be automated using tools such as `npm-check-updates` (ncu), which can query the registry for available versions and update the local `package.json` files accordingly, reducing manual errors.10

The required metadata transformation is summarized below:

Package Metadata Transformation Example

|**Field**|**Current State (Git Distribution)**|**Required State (GitHub NPM)**|**Significance**|
|---|---|---|---|
|`name`|`com.tglgames.core`|`@tglGames-Plugins/com.tglgames.core`|Enables GitHub NPM scoping and UPM routing.3|
|`dependencies` (internal)|`"com.tglgames.util": "git+url..."`|`"@tglGames-Plugins/com.tglgames.util": "1.0.0"`|Switches to automated, versioned dependency resolution.|

---

## PHASE III: Unity Consumption Setup

This phase instructs developers on configuring their consuming Unity projects to successfully fetch the newly published packages using the scoped registry mechanism.

### 6. Configuring Unity Package Manager (UPM) for Public Packages

The Unity project's `manifest.json` file must be modified to instruct the UPM where to look for packages prefixed with `com.tglgames`.

#### Modification of the Project `Packages/manifest.json`

Developers must locate the `Packages/manifest.json` file in their Unity project root and add a `scopedRegistries` property. This configuration needs to be applied only once per consuming project to define the custom package source.2

#### Detailed Schema for the `scopedRegistries` Entry

The configuration object must specify the Name, URL, and the relevant Scopes. The `scopes` property is critical; it defines the namespace prefixes that UPM will route to this custom registry.1

While the general GitHub Packages endpoint is `https://npm.pkg.github.com`, it is recommended practice to include the organization scope in the URL to clearly identify the source: `https://npm.pkg.github.com/@tglGames-Plugins`. The `scopes` array must contain the reverse-domain prefix, `com.tglgames`, ensuring that only packages beginning with this identifier are sought from the GitHub registry.

The JSON structure for the `scopedRegistries` entry should be defined as follows:

Unity Project manifest.json Scoped Registry Configuration

|**Field**|**JSON Type**|**Example Value**|**Role/Description**|
|---|---|---|---|
|`name`|String|`TGL Games Plugins`|Display name in the Unity Package Manager UI.2|
|`url`|String|`https://npm.pkg.github.com/@tglGames-Plugins`|The organization’s GitHub NPM registry endpoint.|
|`scopes`|Array|`["com.tglgames"]`|Specifies the reverse-domain prefixes UPM routes to this URL.2|

#### Testing Package Resolution and Installation in UPM

Once the `scopedRegistries` section is configured, the package can be added to the standard `dependencies` block within `manifest.json`.

A crucial distinction must be maintained in the dependency listing: while the published package name is the scoped NPM format (`@tglGames-Plugins/com.tglgames.packagename`), the dependency entry in the consuming project’s `manifest.json` _must_ use the original UPM-style name:

JSON

```
{
  "dependencies": {
    "com.tglgames.packagename": "1.0.0"
  }
}
```

UPM resolves the dependency by first checking the `scopes` array for a matching prefix (`com.tglgames`). If a match is found, UPM routes the request to the specified GitHub URL, allowing the registry to handle the resolution against its stored scoped packages.2

### 7. Addressing Non-Scoped Unity/NPM Packages

Because the configuration restricts routing only to packages starting with `com.tglgames`, UPM's default behavior remains intact for all other dependency types. Packages such as `com.unity.textmeshpro` or generic third-party NPM dependencies are automatically sought from the default Unity or NPM registries, respectively. This scoped approach ensures seamless co-existence and prevents dependency conflicts or unexpected authentication requirements for unrelated packages.1

---

## PHASE IV: Transitioning to Private Visibility and Secure Access Control

This final phase prepares the organization for future security hardening by converting the public registry to private access, necessitating a dedicated authentication mechanism for consumption.

### 8. Strategy for Converting Packages to Private Visibility

The GitHub Packages NPM registry supports highly configurable access control, allowing packages to be secured independently of their underlying source code repositories.7

#### GitHub Packages Granular Permissions Management

The NPM registry is one of the registries supporting granular permissions.7 This means that the visibility and access rights of the package itself can be set separately from the permissions of the repository it is linked to.

#### Setting Package Visibility and Access Policies

Once the packages have been successfully deployed, an administrator can use the GitHub Package settings interface to transition the package visibility from Public to Private.

Following this transition, organization administrators must establish explicit access policies:

- **Read Access:** This role should be assigned to specific developer teams or organization members who only need to consume (pull/install) the packages.
    
- **Admin/Write Access:** This highly restricted role should be limited to CI/CD systems or designated administrative personnel responsible for publishing new versions or performing maintenance.7
    

### 9. Implementing PAT-Based Authentication for Private Consumption

Once a package is marked Private, every developer attempting to consume it via UPM must authenticate their package manager client.

#### Required PAT Scopes for Private Reading

A crucial security practice is to generate a new, separate PAT specifically for reading private packages. This PAT must be granted only the minimum required scope: `read:packages`.3 Using a read-only token maintains the principle of least privilege, preventing accidental or malicious use of the publishing token (which holds

`write:packages`) during day-to-day development.

#### Configuring User-Level Authentication (`.upmconfig.toml`)

Unity Package Manager (UPM) does not typically rely on the project's local `.npmrc` file for authentication during package installation. Instead, UPM requires credentials to be stored in a user-specific configuration file, `.upmconfig.toml`, located outside the project root.11

The file location is typically in the user's profile directory (e.g., `~/.upmconfig.toml` on macOS/Linux, or `C:\Users\<username>\.upmconfig.toml` on Windows).

This file maps the scoped registry URL to the dedicated read PAT, ensuring that credentials persist across all Unity projects managed by that developer without exposing the PAT within source control.12 If a developer's PAT expires or is revoked, their ability to fetch packages will immediately cease, highlighting the need for careful lifecycle management of these access tokens.

The configuration structure within `.upmconfig.toml` must be precise:

`.upmconfig.toml` Private Consumption Configuration

|**Key**|**Value**|**File Location**|**Role**|
|---|---|---|---|
|`[npmAuth."https://npm.pkg.github.com/@tglGames-Plugins"]`|_Section Header_|User Profile Directory|Specifies the target registry endpoint for authentication.12|
|`token`|`<Read:packages PAT>`|User Profile Directory|Stores the authenticated Personal Access Token (PAT) for access.12|
|`email`|`<Your Email Address>`|User Profile Directory|Required metadata for UPM authentication.12|
|`alwaysAuth`|`true`|User Profile Directory|Ensures UPM always attempts authentication for this scoped registry.12|

This implementation establishes two necessary, yet separate, authentication vectors: the NPM CLI uses the `write:packages` PAT for publishing, and UPM uses the `read:packages` PAT stored in the user profile for consumption. This separation is fundamental to maintaining organizational security posture.

## Conclusions and Recommendations

The successful migration of the `com.tglgames.*` packages from Git URL distribution to a versioned, public GitHub Packages NPM registry is achievable through a four-phased approach focusing on strict metadata standardization, scoped authentication, and dependency refactoring. The complexity of the project is primarily derived from reconciling UPM's reverse-domain naming with GitHub's scoped NPM format and managing separate authentication mechanisms for publishing and consuming packages.

### Actionable Recommendations for Workflow Hardening

1. **Standardize Scoping and Naming:** The use of the dual-scope naming convention (`@tglGames-Plugins/com.tglgames.packagename`) is non-negotiable for interoperability between GitHub Packages and UPM’s routing mechanism.
    
2. **Strict Sequencing for Dependency Refactoring:** Ensure all core packages are published as version `1.0.0` (or greater) _before_ attempting to update dependent `package.json` files to reference version numbers instead of Git URLs.
    
3. **Future CI/CD Migration:** While initial publishing requires a PAT with `write:packages` scope, the long-term, most secure strategy for automated publishing workflows involves migrating to GitHub Actions. Workflows associated with the package’s repository can leverage the built-in `GITHUB_TOKEN` for authentication, which automatically receives the necessary access permissions and removes the reliance on a long-lived user PAT for continuous integration.3
    
4. **Long-Term Access Management:** While user-specific PATs (`read:packages`) stored in `.upmconfig.toml` are the immediate solution for private consumption, for enterprise environments, consideration should be given to migrating long-lived registry access to a GitHub App. GitHub recommends using GitHub Apps instead of long-lived user PATs for organizational or permanent integrations, offering superior security and management controls over credential lifecycles.6


> [!Note]
> The Following section of the document is from Deep Seek, I shall update or edit if needed.
# Suggestion from DeepSeek
Based on your request, I've researched how to set up a scoped registry for your Unity packages using GitHub Packages. Here is a detailed, step-by-step guide to help you through the process.

## 📖 Understanding GitHub Packages for Unity

GitHub Packages is a package hosting service that allows you to publish private or public packages. It supports various package registries, including the **npm registry**, which the Unity Package Manager can use with some specific configuration.

The core idea is to configure your Unity project to use a **Scoped Registry**. This tells the Unity Package Manager to look for packages with specific naming scopes (like `com.tglgames`) in your custom registry (GitHub Packages) instead of the default Unity registry.

> **Important Note:** According to Unity's team, using GitHub Packages Registry with Unity Package Manager has limitations and is not officially supported. A key limitation is that packages will not appear in the Package Manager's "All" tab; you must install them manually by editing your project's `manifest.json` file.

## 🔧 Step-by-Step Setup Guide

### Step 1: Modify Your Package's `package.json`

To be compatible with the workaround for Unity, you must format your `package.json` differently from a standard npm package.

1.  **Do Not Use an NPM Scope in the Name:** Your package name should not include the `@username/` or `@org/` prefix typically used for scoped npm packages.
    *   **Correct:** `"name": "com.tglgames.tgl-file-read-write"`
    *   **Incorrect:** `"name": "@tglGames-Plugins/com.tglgames.tgl-file-read-write"`

2.  **Add the `publishConfig` Field:** This is a crucial part of the workaround. It directs the publishing tool to your specific GitHub Packages registry URL. Add the following to your `package.json`, replacing `@USER` with your GitHub organization name (`@tglGames-Plugins`).
    ```json
    "publishConfig": {
      "registry": "https://npm.pkg.github.com/@tglGames-Plugins"
    },
    ```

    Here is a complete example of how your `package.json` for `com.tglgames.tgl-file-read-write` should look:
    ```json
    {
      "name": "com.tglgames.tgl-file-read-write",
      "version": "1.0.0",
      "displayName": "TGL File Read Write",
      "description": "This is an file read write package...",
      "publishConfig": {
        "registry": "https://npm.pkg.github.com/@tglGames-Plugins"
      },
      "author": {
        "name" : "Rishabh Jain",
        "url" : "https://tglblog.vercel.app/"
      },
      "unity": "6000.0",
      "dependencies": {
        "com.tglgames.tgl-file-utility": "1.0.0"
      }
    }
    ```
    **Note on Dependencies:** You can no longer use Git URLs for dependencies within your `package.json` if those dependencies are also being published to your new registry. You must use the version number, and ensure the dependent package (e.g., `com.tglgames.tgl-file-utility`) is also published to your scoped registry.

### Step 2: Publish Your Package to GitHub Packages

You can publish your package using the npm CLI or by automating the process with GitHub Actions.

#### **Option A: Publish Using the npm CLI**

1.  **Create a Personal Access Token:** You need a classic Personal Access Token to authenticate. Go to your GitHub **Settings** > **Developer settings** > **Personal access tokens** > **Tokens (classic)** and generate a new token with the `read:packages` and `write:packages` scopes.
2.  **Authenticate with npm:** Run the following command in your terminal, replacing `USERNAME` with your GitHub username and `TOKEN` with the token you just created:
    ```bash
    npm login --scope=@tglGames-Plugins --auth-type=legacy --registry=https://npm.pkg.github.com
    > Username: USERNAME
    > Password: TOKEN
    ```
3.  **Publish the Package:** Navigate to your package's directory and run:
    ```bash
    npm publish
    ```

#### **Option B: Publish Using GitHub Actions (Recommended)**

This method automatically publishes your package whenever you create a new release.

1.  In your repository, create a file at `.github/workflows/release-package.yml`.
2.  Paste the following YAML content into the file:
    ```yaml
    name: Node.js Package
    on:
      release:
        types: [created]
    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v5
          - uses: actions/setup-node@v4
            with:
              node-version: 20
          - run: npm ci
          # Add your project's build or test commands here
      publish-gpr:
        needs: build
        runs-on: ubuntu-latest
        permissions:
          packages: write
          contents: read
        steps:
          - uses: actions/checkout@v5
          - uses: actions/setup-node@v4
            with:
              node-version: 20
              registry-url: https://npm.pkg.github.com/
          - run: npm ci
          - run: npm publish
            env:
              NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    ```
3.  Commit and push this file. When you next create a release in your GitHub repository, this workflow will run and publish your package.

### Step 3: Configure a Unity Project to Use Your Scoped Registry

To install packages from your registry, you must configure each Unity project that will use them.

1.  Open your Unity project.
2.  Go to **Edit** > **Project Settings** > **Package Manager**.
3.  In the **Scoped Registries** section, add a new registry with the following details:
    *   **Name:** `TGL Games Plugins` (This can be any descriptive name).
    *   **URL:** `https://npm.pkg.github.com/@tglGames-Plugins` (This *must* match the URL in your `publishConfig`).
    *   **Scope(s):** `com.tglgames` (This tells Unity to look for any package starting with this prefix in your registry).
4.  Click **Apply**.

### Step 4: Install Packages in Your Unity Project

Due to the mentioned limitation, you cannot install these packages directly from the Package Manager UI. You must manually add them to your project's `manifest.json` file.

1.  Open the `manifest.json` file located in your project's `Packages` folder.
2.  Ensure your `scopedRegistries` configuration is present. It should look something like this:
    ```json
    {
      "scopedRegistries": [
        {
          "name": "TGL Games Plugins",
          "url": "https://npm.pkg.github.com/@tglGames-Plugins",
          "scopes": [
            "com.tglgames"
          ]
        }
      ],
      "dependencies": {
        "com.unity.collab-proxy": "2.0.4",
        ...
      }
    }
    ```
3.  In the `dependencies` block, add your package and its version. The version must match exactly what you published.
    ```json
    "dependencies": {
      "com.unity.collab-proxy": "2.0.4",
      "com.tglgames.tgl-file-read-write": "1.0.0",
      ...
    }
    ```
4.  Save the file. Unity will automatically resolve the new dependency and install the package from your GitHub Packages registry.

## 🔑 Authentication for Other Users

**GitHub Packages always requires authentication, even for public packages**. This means anyone who wants to install your packages will need to provide credentials.

1.  They must create a **classic Personal Access Token** with at least the `read:packages` permission.
2.  They need to create an `.npmrc` file in the root of their Unity project (next to the `Assets` and `Packages` folders) with the following content, replacing `TOKEN` with their actual token:
    ```
    //npm.pkg.github.com/:_authToken=TOKEN
    ```
    > **Important Security Note:** They should add this `.npmrc` file to their `.gitignore` to avoid accidentally committing their token to a public repository.

## ⚠️ Summary of Limitations

- **No UI Discoverability:** Your packages will not show up in the Package Manager UI's "All" tab. Installation is done by manually editing the `manifest.json` file.
- **Authentication Required:** All users must provide a Personal Access Token, even for public packages.
- **Dependency Management:** You cannot use Git URLs to define dependencies between packages in your `package.json`. All packages that depend on each other must be published to the same scoped registry and referenced by version number.

This setup creates a centralized, version-controlled repository for your Unity packages. While it has limitations, it provides a more robust and scalable solution for managing your plugins compared to using direct Git URLs.

I hope this detailed guide helps you get started. If you run into any specific issues during the setup, feel free to ask.



---

[^1]:
[^2]:

