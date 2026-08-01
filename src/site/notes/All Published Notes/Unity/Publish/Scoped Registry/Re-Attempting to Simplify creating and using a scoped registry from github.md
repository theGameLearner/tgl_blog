---
{"dg-publish":true,"permalink":"/all-published-notes/unity/publish/scoped-registry/re-attempting-to-simplify-creating-and-using-a-scoped-registry-from-github/"}
---

created: 2026-07-23
updated: 2026-07-26

### General Idea
I have been working on homelab experimentation for sometime and want to ensure I do everything correctly when trying to create a new github package for publishing. I created a package for tutorial handler, this will allow us to create a tutorial with 
- a blocker image where we can control the color and alpha
- cutout for any rect transform that we want to highlight
- rebase to reparent an object so we can interact with a button without having sharp edges in the cutout.
Now I will try to publish it in github as a package.

### Current Situation

#### Unity folder
For this, I made a folder inside my project and divided it into two folders and assembly definitions:
- `Runtime`: has all prefabs, scripts and UI for using this package
- `Sample`: has an example scene showing it's use using test case.

directory content:
```bash
bash-5.2$pwd
/home/thegamelearner/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler
bash-5.2$ls -al
total 24
drwxr-xr-x  4 thegamelearner thegamelearner 4096 Jul 19 16:02 .
drwxr-xr-x 12 thegamelearner thegamelearner 4096 Jul 23 19:17 ..
drwxr-xr-x  7 thegamelearner thegamelearner 4096 Jul 23 20:01 Runtime
-rw-rw-r--  1 thegamelearner thegamelearner  172 Jul 19 15:52 Runtime.meta
drwxr-xr-x  3 thegamelearner thegamelearner 4096 Jul 23 20:02 Sample
-rw-rw-r--  1 thegamelearner thegamelearner  172 Jul 19 16:02 Sample.meta
bash-5.2$
```

#### Github Account
##### Repository
I have not made a github project to store the raw files for this project.

##### PAT
The PAT from Github is generated and available : `ghp_****`
We can test it with:
```sh
TOKEN="ghp_****"

echo "--- User Check ---"
curl -s -H "Authorization: Bearer $TOKEN" https://api.github.com/user | grep -E '"login"|"message"'

echo "--- Organization Check ---"
curl -s -H "Authorization: Bearer $TOKEN" https://api.github.com/user/memberships/orgs/tglGames-Plugins | grep -E '"state"|"role"|"message"'
```

Example for user 'https://github.com/tglGames' with organization 'https://github.com/tglGames-Plugins':
```bash
bash-5.2$ TOKEN="ghp_******"
bash-5.2$curl -s -H "Authorization: Bearer $TOKEN" https://api.github.com/user | grep -E '"login"|"message"'
  "login": "tglGames",
bash-5.2$curl -s -H "Authorization: Bearer $TOKEN" https://api.github.com/user/memberships/orgs/tglGames-Plugins | grep -E '"state"|"role"|"message"'
  "state": "active",
  "role": "admin",
bash-5.2$
```

##### Organization
The organization has the packages set to public:
Open Settings for Organizations as user(`https://github.com/settings/organizations`), open settings inside the Organization.
![github user settings.png](/img/user/All%20Published%20Notes/Unity/Publish/Scoped%20Registry/Images/github%20user%20settings.png)
Open Settings inside the organization settings, make sure the package creation is public.
![github organization package settings.png](/img/user/All%20Published%20Notes/Unity/Publish/Scoped%20Registry/Images/github%20organization%20package%20settings.png)



### Define the scope in project
#### package json file
In our Unity folder, we need to define the package name and scope for the package, this is achieved by a json file called `package.json`:
```bash
bash-5.2$ls -al
total 24
drwxr-xr-x  4 thegamelearner thegamelearner 4096 Jul 19 16:02 .
drwxr-xr-x 12 thegamelearner thegamelearner 4096 Jul 23 19:17 ..
drwxr-xr-x  7 thegamelearner thegamelearner 4096 Jul 23 20:01 Runtime
-rw-rw-r--  1 thegamelearner thegamelearner  172 Jul 19 15:52 Runtime.meta
drwxr-xr-x  3 thegamelearner thegamelearner 4096 Jul 23 20:02 Sample
-rw-rw-r--  1 thegamelearner thegamelearner  172 Jul 19 16:02 Sample.meta
bash-5.2$fresh package.json

A new version of fresh is available: 0.2.17 -> 0.4.4
Update with: npm update -g @fresh-editor/fresh-editor

bash-5.2$cat -n package.json
     1  {
     2    "name": "com.tglgames.tgl-tutorial-manager",
     3    "version": "1.0.0",
     4    "displayName": "TGL Tutorial Manager",
     5    "description": "This is an Tutorial Manager package, use it to create tutorials in game",
     6    "author": {
     7      "name" : "Rishabh Jain",
     8      "email" : "thegamelearner@gmail.com",
     9      "url" : "https://tglblog.vercel.app/"
    10    },
    11    "publishConfig": { "registry": "https://npm.pkg.github.com/@tglGames-Plugins" },
    12    "unity": "2019.1",
    13    "unityRelease": "0b5",
    14    "documentationUrl": "",
    15    "changelogUrl": "",
    16    "licensesUrl": "https://opensource.org/license/apache-2-0",
    17    "dependencies": { },
    18    "keywords": [
    19      "Tutorial Handler",
    20      "TGL"
    21    ]
    22  }
    23
bash-5.2$ls -al
total 28
drwxr-xr-x  4 thegamelearner thegamelearner 4096 Jul 23 21:50 .
drwxr-xr-x 12 thegamelearner thegamelearner 4096 Jul 23 19:17 ..
-rw-rw-r--  1 thegamelearner thegamelearner  655 Jul 23 21:50 package.json
drwxr-xr-x  7 thegamelearner thegamelearner 4096 Jul 23 20:01 Runtime
-rw-rw-r--  1 thegamelearner thegamelearner  172 Jul 19 15:52 Runtime.meta
drwxr-xr-x  3 thegamelearner thegamelearner 4096 Jul 23 20:02 Sample
-rw-rw-r--  1 thegamelearner thegamelearner  172 Jul 19 16:02 Sample.meta
bash-5.2$
```
By using `publishConfig` we are ensuring to retain the scope in our account without having to name the package in a format preferred by npm[^2].
see also: [[#Fixing the package release (only for first time)]] 
> [!Warning]
> As observed in previous runs, if a raw package without an explicit scope name fails to index initially on GitHub Packages UI, temporarily setting `"name": "@tglGames-Plugins/com.tglgames.tgl-tutorial-manager"` to publish, deleting it, and re-publishing with the clean name `com.tglgames.tgl-tutorial-manager` forces GitHub to register the scoping link).

> [!Note]
> Ensure we start with version 1.0.0 as the minimum version if we want to use npm to find it

##### Add text mesh pro as a dependency
My runtime folder uses `Unity.TextMeshPro` assembly definition for UI text.
So we need to add the dependency in the project's package.json as well.

As TextMeshPro was merged to `unity.ugui` in 2023, we can use that.
Update the json to include the dependency:
```json
{
  "name": "com.tglgames.tgl-tutorial-manager",
  "version": "1.0.0",
  "displayName": "TGL Tutorial Manager",
  "description": "This is an Tutorial Manager package, use it to create tutorials in game",
  "author": {
    "name" : "Rishabh Jain",
    "email" : "thegamelearner@gmail.com",
    "url" : "https://tglblog.vercel.app/"
  },
  "publishConfig": { "registry": "https://npm.pkg.github.com/@tglGames-plugins" },
  "unity": "2023.2",
  "unityRelease": "0b5",
  "documentationUrl": "",
  "changelogUrl": "",
  "licensesUrl": "https://opensource.org/license/apache-2-0",
  "dependencies": {
    "com.unity.ugui": "2.0.0"
  },
  "keywords": [
    "Tutorial Handler",
    "TGL"
  ]
}
```

#### Extra files
We also need:
- a `README.md` for github page
- a `CHANGELOG.md` for the changes we do
- a `Documentation` directory for any notes, images, etc we want to add to any markdown files.
- a `License.md` for license we use (Apache License in my case)
- a `.npmrc` file which will define the registry to use for specific scopes.

As the rest of the files should be self explanatory, we will add `.npmrc` as a focus point:
```
bash-5.2$ls -al
total 36
drwxr-xr-x  4 thegamelearner thegamelearner 4096 Jul 23 21:59 .
drwxr-xr-x 12 thegamelearner thegamelearner 4096 Jul 23 19:17 ..
-rw-rw-r--  1 thegamelearner thegamelearner   92 Jul 23 21:59 .npmrc
-rw-rw-r--  1 thegamelearner thegamelearner  655 Jul 23 21:50 package.json
-rw-rw-r--  1 thegamelearner thegamelearner  158 Jul 23 21:54 package.json.meta
drwxr-xr-x  7 thegamelearner thegamelearner 4096 Jul 23 20:01 Runtime
-rw-rw-r--  1 thegamelearner thegamelearner  172 Jul 19 15:52 Runtime.meta
drwxr-xr-x  3 thegamelearner thegamelearner 4096 Jul 23 20:02 Sample
-rw-rw-r--  1 thegamelearner thegamelearner  172 Jul 19 16:02 Sample.meta
bash-5.2$cat -n .npmrc
     1  @tglGames-Plugins:registry=https://npm.pkg.github.com/
     2  registry=https://registry.npmjs.org/
bash-5.2$
```
Here, we are telling the system that if the scope is `@tglGames-Plugins`, use `https://npm.pkg.github.com/`(github npm registry) as the registry instead of default registry, but if the scope is something different, use the default registry of npm(`https://registry.npmjs.org/`).

see also [[My Step by Step Guide#3. Create and configure `.npmrc`|configure `.npmrc`]] in case the `.npmrc` file does not work.

The rest of the files can be made and updated as needed.

### Create github Repository
Create a repository which we will use as to host the tutorial package:
- Open repositories package: `https://github.com/orgs/tglGames-Plugins/repositories`
- click on 'New Repository'
- Fill in the details
- Create Repository
- The repository is created `https://github.com/tglGames-Plugins/tgl-tutorial-manager`.
details filled:
![github new repository.png](/img/user/All%20Published%20Notes/Unity/Publish/Scoped%20Registry/Images/github%20new%20repository.png)

#### CLI registry authentication
The terminal that is supposed to upload packages needs to login to npm registry within the relevant scope.
```sh

Sun Jul 26, 05:29:39 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ npm login --scope=@tglgames-plugins --auth-type=legacy --registry=https://npm.pkg.github.com
npm notice Log in on https://npm.pkg.github.com/
Username: tglgames
Password: 
Logged in to scope @tglgames-plugins on https://npm.pkg.github.com/.

Sun Jul 26, 05:29:53 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ 
```
it will ask for username and password, use `tglgames` and the pat `ghp_***`

#### Push the project files
Initialize and push your package source code:
```sh

Sun Jul 26, 05:29:53 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ ls -al
total 56
drwxr-xr-x  4 thegamelearner thegamelearner  4096 Jul 26 17:23 .
drwxr-xr-x 12 thegamelearner thegamelearner  4096 Jul 23 19:17 ..
-rw-rw-r--  1 thegamelearner thegamelearner 11358 Jul 26 17:23 License.md
-rw-rw-r--  1 thegamelearner thegamelearner    92 Jul 23 21:59 .npmrc
-rw-rw-r--  1 thegamelearner thegamelearner   655 Jul 23 21:50 package.json
-rw-rw-r--  1 thegamelearner thegamelearner   158 Jul 23 21:54 package.json.meta
-rw-rw-r--  1 thegamelearner thegamelearner  1770 Jul 23 22:26 README.md
-rw-rw-r--  1 thegamelearner thegamelearner   158 Jul 23 22:21 README.md.meta
drwxr-xr-x  7 thegamelearner thegamelearner  4096 Jul 23 20:01 Runtime
-rw-rw-r--  1 thegamelearner thegamelearner   172 Jul 19 15:52 Runtime.meta
drwxr-xr-x  3 thegamelearner thegamelearner  4096 Jul 23 20:02 Sample
-rw-rw-r--  1 thegamelearner thegamelearner   172 Jul 19 16:02 Sample.meta

Sun Jul 26, 05:30:28 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git init
Initialized empty Git repository in /home/thegamelearner/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler/.git/

Sun Jul 26, 05:31:29 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git add .

Sun Jul 26, 05:31:35 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git commit -m "feat: Initial package setup for TGL Tutorial Manager"
[main (root-commit) 30f8bbe] feat: Initial package setup for TGL Tutorial Manager
 72 files changed, 7036 insertions(+)
 create mode 100644 .npmrc
 create mode 100644 License.md
 ...
 
 create mode 100644 package.json
 create mode 100644 package.json.meta

Sun Jul 26, 05:31:45 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git 

Sun Jul 26, 05:32:51 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git branch -M main

Sun Jul 26, 05:32:58 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ 

Sun Jul 26, 05:33:03 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ ssh -T git@github.com
git@github.com: Permission denied (publickey).

Sun Jul 26, 05:38:02 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ ssh -T git@github.com-tglGames-Plugins
Hi tglGames! You\'ve successfully authenticated, but GitHub does not provide shell access.

Sun Jul 26, 05:45:44 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git remote set-url origin git@github.com-tglGames-Plugins:tglGames-Plugins/tgl-tutorial-manager.git

Sun Jul 26, 05:46:02 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git push -u origin main --force
Enumerating objects: 92, done.
Counting objects: 100% (92/92), done.
Delta compression using up to 24 threads
Compressing objects: 100% (92/92), done.
Writing objects: 100% (92/92), 193.39 KiB | 929.00 KiB/s, done.
Total 92 (delta 27), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (27/27), done.
To github.com-tglGames-Plugins:tglGames-Plugins/tgl-tutorial-manager.git
 + 156f9ee...30f8bbe main -> main (forced update)
branch 'main' set up to track 'origin/main'.

Sun Jul 26, 05:46:11 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ 
```

Now the package is uploaded to the github.

### Publish the package to this repository
##### Publish the package in registry
We can publish using `npm publish --access public`:
```sh

Sun Jul 26, 05:46:22 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ npm publish --access public
npm notice 
npm notice package: com.tglgames.tgl-tutorial-manager@1.0.0
npm notice === Tarball Contents === 
npm notice 11.4kB License.md                       
...
                                                                   
npm notice 158B   package.json.meta                                                                    
npm notice === Tarball Details === 
npm notice name:          com.tglgames.tgl-tutorial-manager          
npm notice version:       1.0.0                                      
npm notice filename:      com.tglgames.tgl-tutorial-manager-1.0.0.tgz
npm notice package size:  196.0 kB                                   
npm notice unpacked size: 403.1 kB                                   
npm notice shasum:        e22aa94725e2839dd5e3a3e10c65df62055ecdf8   
npm notice integrity:     sha512-Aii9Dq3XacEEn[...]740F15SHcKxLg==   
npm notice total files:   71                                         
npm notice 
npm notice Publishing to https://npm.pkg.github.com/@tglGames-Plugins with tag latest and public access
+ com.tglgames.tgl-tutorial-manager@1.0.0

Sun Jul 26, 05:51:56 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ 
```

##### Tag the repository package

We can tag the release:
```sh
Sun Jul 26, 05:51:56 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git tag v1.0.0

Sun Jul 26, 05:52:53 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git push origin v1.0.0
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com-tglGames-Plugins:tglGames-Plugins/tgl-tutorial-manager.git
 * [new tag]         v1.0.0 -> v1.0.0

Sun Jul 26, 05:52:59 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ 
```

#### Fixing the package release (only for first time)
Due to naming conflict in Unity and NPM naming conventions, the package appears as published but is not found when searching. To avoid this, we will make a release on version `1.0.1` with the name changed to `"@tglGames-Plugins/com.tglgames.tgl-tutorial-manager"` and then release `1.0.2` with the name changed back to `"name": "com.tglgames.tgl-tutorial-manager",`

Open `package.json` in an editor (vs code or text editor) and release it with version changes as shown in terminal output:
```sh
Sun Jul 26, 08:54:45 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ head -n 3 package.json
     1  {
     2    "name": "@tglGames-Plugins/com.tglgames.tgl-tutorial-manager",
     3    "version": "1.0.1",
Sun Jul 26, 08:55:29 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ npm publish
npm notice 
npm notice package: @tglGames-Plugins/com.tglgames.tgl-tutorial-manager@1.0.1
npm notice === Tarball Contents === 
npm notice 11.4kB License.md 
...
npm notice === Tarball Details === 
npm notice name:          @tglGames-Plugins/com.tglgames.tgl-tutorial-manager         
npm notice version:       1.0.1                                                       
npm notice filename:      tglGames-Plugins-com.tglgames.tgl-tutorial-manager-1.0.1.tgz
npm notice package size:  196.0 kB                                                    
npm notice unpacked size: 403.1 kB                                                    
npm notice shasum:        9670ee737a2645937a63769b495b118cd86518f0                    
npm notice integrity:     sha512-blPJlKSQjL6uP[...]N8sDaLPDuVrWw==                    
npm notice total files:   71                                                          
npm notice 
npm notice Publishing to https://npm.pkg.github.com/ with tag latest and default access
+ @tglGames-Plugins/com.tglgames.tgl-tutorial-manager@1.0.1

Sun Jul 26, 08:55:32 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ 

Sun Jul 26, 08:56:01 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ head -n 3 package.json
     1  {
     2    "name": "com.tglgames.tgl-tutorial-manager",
     3    "version": "1.0.2",
Sun Jul 26, 08:56:03 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ npm publish
npm notice 
npm notice package: com.tglgames.tgl-tutorial-manager@1.0.2
npm notice === Tarball Contents === 
npm notice 11.4kB License.md
...
npm notice === Tarball Details === 
npm notice name:          com.tglgames.tgl-tutorial-manager          
npm notice version:       1.0.2                                      
npm notice filename:      com.tglgames.tgl-tutorial-manager-1.0.2.tgz
npm notice package size:  196.0 kB                                   
npm notice unpacked size: 403.1 kB                                   
npm notice shasum:        2245c477fd2355597955732e595dcb4afdd438b7   
npm notice integrity:     sha512-dHmAOlnxtNzmv[...]1lkkSbEtwv6MQ==   
npm notice total files:   71                                         
npm notice 
npm notice Publishing to https://npm.pkg.github.com/@tglGames-Plugins with tag latest and default access
+ com.tglgames.tgl-tutorial-manager@1.0.2

Sun Jul 26, 08:56:07 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ 
```

I renamed in an open editor but have used head to show the change made before publishing.
Remember to re-tag the release to v1.0.2 (`git tag v1.0.2`)
```sh

Sun Jul 26, 09:30:07 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git tag -l "v1.*"
v1.0.0

Sun Jul 26, 09:30:13 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git tag -n
v1.0.0          feat: Initial package setup for TGL Tutorial Manager

Sun Jul 26, 09:30:19 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   package.json

no changes added to commit (use "git add" and/or "git commit -a")

Sun Jul 26, 09:32:30 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git add .

Sun Jul 26, 09:32:45 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git commit -m "Bump version to 1.0.2"
[main 06524bb] Bump version to 1.0.2
 1 file changed, 1 insertion(+), 2 deletions(-)

Sun Jul 26, 09:32:46 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git tag -a v1.0.2 -m "Release version 1.0.2"

Sun Jul 26, 09:32:52 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git push origin v1.0.2
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 24 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 457 bytes | 457.00 KiB/s, done.
Total 4 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To github.com-tglGames-Plugins:tglGames-Plugins/tgl-tutorial-manager.git
 * [new tag]         v1.0.2 -> v1.0.2

Sun Jul 26, 09:33:06 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git push --tags
Everything up-to-date

Sun Jul 26, 09:33:12 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ git tag -l
v1.0.0
v1.0.2

Sun Jul 26, 09:33:30 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ 
```

#### Verify the package is published
Open the organization packages page in a browser: "https://github.com/orgs/tglGames-Plugins/packages"
Confirm the new package appears as a published package: "com.tglgames.tgl-tutorial-manager"
Open the package and confirm the version: `com.tglgames.tgl-tutorial-manager@1.0.2`('https://github.com/orgs/tglGames-Plugins/packages/npm/package/com.tglgames.tgl-tutorial-manager')

At the bottom of this page, there is button to connect the package to a repository, it won't be an issue, but you can connect the package to repository for easy reference.

### Download or test the package

Create a new project in which we will try to download and use the package:
- create new package
- Unity version: 6000.2.6f2

#### Add toml file for access
Unity Package Manager (UPM) does not typically rely on the project's local `.npmrc` file for authentication during package installation. Instead, UPM requires credentials to be stored in a user-specific configuration file, `.upmconfig.toml`.

Add toml file in the project or in the host machine, a toml file will define the table for npm authentication with the token(pat) and the email(the user id) of the user. It is like user name and password to authenticate access with github, even if the released package was public.

##### Add toml for the host machine
If the packages are to be used in a company development machine, where we need the user always accessible, we can add the toml file to the machine to avoid writing it every time.
the file needs to be stored in standard path:
- **Linux/macOS:** `~/.upmconfig.toml` (located in `/home/<your-user>/.upmconfig.toml`)
- **Windows:** `%USERPROFILE%\.upmconfig.toml` (located in `C:\Users\<your-user>\.upmconfig.toml`)

Assuming Linux, create `~/.upmconfig.toml` on the host machine:
```toml
[npmAuth."https://npm.pkg.github.com/@tglGames-Plugins"] 
token = "ghp_TESTERS_OWN_READ_PACKAGES_PAT"
email = "it.rishabh.jain@gmail.com"
alwaysAuth = true
```

##### Add toml for the project

> [!Warning]
> If you want to use local `toml` file, the Unity editor or Unity Hub has to be started from the terminal where the export was set (unless you add this to bashrc). If this is not added, you may not be able to read the packages.

If you are working on a personal project and need access to the scoped registry/package, add the toml to the project and not the whole machine:
Let's add the toml to the Assets folder of the project under a directory called `AccessConfig` named `upmconfig.toml`:
```toml
[npmAuth."https://npm.pkg.github.com/@tglGames-Plugins"]
token = "ghp_TESTERS_OWN_READ_PACKAGES_PAT"
email = "it.rishabh.jain@gmail.com"
alwaysAuth = true
```

As this project does not know where the custom toml file exists, we need to export it and then UPM will use this to read packages
so we open unity from terminal now:
```sh
$ export UPM_USER_CONFIG_FILE="/path/to/project/projectUPMRegistries.toml"
$ "/Path/To/Unity/Editor/Unity" -projectPath "/path/to/project"
```

so, if the project is at: `/home/thegamelearner/Documents/Unity Projects/TestTutorial/`
the toml file is at: `/home/thegamelearner/Documents/Unity Projects/TestTutorial/Assets/AccessConfig/upmconfig.toml`
and the unity we are using is at: `/home/thegamelearner/Unity/Hub/Editor/6000.2.6f2/Editor/Unity`

we will open Unity as :
```sh
$ export UPM_USER_CONFIG_FILE="/home/thegamelearner/Documents/Unity Projects/TestTutorial/Assets/AccessConfig/upmconfig.toml" 
$ 
$ "/home/thegamelearner/Unity/Hub/Editor/6000.2.6f2/Editor/Unity" -projectPath "/home/thegamelearner/Documents/Unity Projects/TestTutorial/"
```

Log:
```sh
gkFLUvqoeMS4ThWyl0Gt0TrQ it.rishabh.jain@google.com (RSA)

Sun Jul 26, 06:36:05 PM "~"|
thegamelearner@thegamelearner-MS-7E12:$ export UPM_USER_CONFIG_FILE="/home/thegamelearner/Documents/Unity Projects/TestTutorial/Assets/AccessConfig/upmconfig.toml" "/home/thegamelearner/Unity/Hub/Editor/6000.2.6f2/Editor/Unity" -projectPath "/home/thegamelearner/Documents/Unity Projects/TestTutorial/"
bash: export: `/home/thegamelearner/Unity/Hub/Editor/6000.2.6f2/Editor/Unity': not a valid identifier
bash: export: `-projectPath': not a valid identifier
bash: export: `/home/thegamelearner/Documents/Unity Projects/TestTutorial/': not a valid identifier

Sun Jul 26, 06:36:06 PM "~"|
thegamelearner@thegamelearner-MS-7E12:$ export UPM_USER_CONFIG_FILE="/home/thegamelearner/Documents/Unity Projects/TestTutorial/Assets/AccessConfig/upmconfig.toml"

Sun Jul 26, 06:39:16 PM "~"|
thegamelearner@thegamelearner-MS-7E12:$ "/home/thegamelearner/Unity/Hub/Editor/6000.2.6f2/Editor/Unity" -projectPath "/home/thegamelearner/Documents/Unity Projects/TestTutorial/"


```

#### Test in Unity
##### Test by downloading as git package (Optional)
If you want to download the package as github project, we can download the project as git package instead of using scoped registry.

After Unity is opened, we tried to download the package using git URL : `https://github.com/tglGames-Plugins/tgl-tutorial-manager.git`
It successfully downloads the package.
###### Script test
We can now create a new script(`TestTutorial.cs`) to see if we can access the scripts made in the package:
```cs
using tglGames.tutorial_manager.tgl_tutorial_handler;  
using UnityEngine;  
  
public class TestTutorial : MonoBehaviour  
{  
    // Start is called once before the first execution of Update after the MonoBehaviour is created  
    void Start()  
    {  
        TutorialHandler.HideTutorialEvent?.Invoke();  
    }  
}
```

> [!Note]
> Remember that to use git url option, `https://github.com/tglGames-Plugins/tgl-tutorial-manager.git` will work but `https://github.com/tglGames-Plugins/tgl-tutorial-manager` will fail

Next we will try to download the package using NPM or scoped registry

##### Test by downloading as scoped registry package
As I added the file in Assets folder under `Assets/AccessConfig/upmconfig.toml`, we will open using the method explained in [[#Add toml for the project]] section.

Open the project
```sh
"/home/thegamelearner/Unity/Hub/Editor/6000.2.6f2/Editor/Unity" -projectPath "/home/thegamelearner/Documents/Unity Projects/TestTutorial/"
```

If the project has the package from git, remove it. Also remove any reference to this package in your script.

###### Add scope registry 
You can use GUI (Unity editor) or manifest.json as needed.

**Add scope registry using Unity menu**
Now, add this scope registry using GUI:
- Edit -> Project Settings
- In 'Project Settings' window, on left side open 'Package Manager'
- Add a new scoped registry(if not already added)
	- Name: `tglGames-Plugins`
	- Url: `https://npm.pkg.github.com/@tglGames-Plugins`
	- Scopes: `com.tglgames`
![unity add scoped registry.png](/img/user/All%20Published%20Notes/Unity/Publish/Scoped%20Registry/Images/unity%20add%20scoped%20registry.png)

**Add the Scope registry in manifest.json**
Now we will add the scope registry in the manifest.json file:
- Open the `manifest.json` file in "/home/thegamelearner/Documents/Unity Projects/TestTutorial/Packages/manifest.json"
- Add the scope registry data in the file:
![adding scope registry in unity manifest.png](/img/user/All%20Published%20Notes/Unity/Publish/Scoped%20Registry/Images/adding%20scope%20registry%20in%20unity%20manifest.png)

Ignore if you get error about searching for repositories:
```
[Package Manager Window] Error searching for packages.
Unable to perform online search:
  Request [GET https://npm.pkg.github.com/@tglGames-Plugins/-/v1/search?text=com.tglgames&from=0&size=250] failed with status code [405]
UnityEditor.EditorApplication:Internal_CallUpdateFunctions () (at /home/bokken/build/output/unity/unity/Editor/Mono/EditorApplication.cs:384)
```
This happens because we do not have search facility inside github package registry.

###### Add the package
You can use GUI (Unity editor) or manifest.json as needed.

**Add the package using GUI**
- Window -> Package Management -> Package Manager
- You may or may not change the registry on left panel to tglGames-Plugins
	- If you used something with search functionality, not github, you will see the package listed here.
	- In Unity, it shows "No items to display" if no other package from this scoped registry was added
- Use `Add(+)` on top left
- use "Install package by name"
- add the name and version
	- name: `com.tglgames.tgl-tutorial-manager`
	- version: `1.0.2`
- Install

> [!Note]
> GitHub Packages explicitly denies/does not support NPM's search API (`/-/v1/search`). Because of this, Unity cannot auto-list or search packages hosted on GitHub inside the Package Manager window.

**Add the package using manifest.json**
If the packages to install are general packages that you want to install for the project, then you can add it to `manifest.json` directly.
Do not add it to `manifest.json` for initial testing, as we want to try and add to find any issues. Add old packages that are tested here.
- open "/home/thegamelearner/Documents/Unity Projects/TestTutorial/Packages/manifest.json"
- Add the package as a dependency: `"com.tglgames.tgl-tutorial-manager": "1.0.2",`
![adding package in manifest json file.png](/img/user/All%20Published%20Notes/Unity/Publish/Scoped%20Registry/Images/adding%20package%20in%20manifest%20json%20file.png)

If done correctly, we can add a script like we did in [[#Script test]] to confirm the package was successfully loaded.
###### Failure Handling
In case of failure, try downloading another package from same scoped registry, for example: `com.tglgames.tgl-fsm`.
- If the other package can be installed and not `com.tglgames.tgl-tutorial-manager`, this means that the package is not available on the registry.
- If both fails, check on all changes done on user end
- If you opened using unity hub, retry by opening the project using the terminal.








---

[^1]: https://docs.unity3d.com/6000.5/Documentation/Manual/CustomPackages.html
[^2]: [okcompute_unity](https://discussions.unity.com/u/okcompute_unity)
[^3]: 


