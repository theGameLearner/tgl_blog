---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/004-users-and-groups-in-proxmox/"}
---

created: 2026-03-23
updated: 2026-03-25

I am having confusion in creation and management of users and groups in Proxmox (Linux) so this document will help me remember the basic commands.

### listing Users and Groups in base Linux
List all groups in Ubuntu: `getent group` : '`groupname:x:GID:`'
List all users in Ubuntu: `getent passwd` : '`username:x:UID:GID:Comment:HomeDirectory:LoginShell`'

### Creating Users and Groups in Linux
#### high level (without specifying id)
**Proxmox**:
- Create a group: `pveum groupadd programmers -comment "Dev Team"`
	- Here, `pveum` (Proxmox VE User Manager) is a command-line tool used in Proxmox Virtual Environment to manage users, groups, roles, and access control lists (ACLs).
	- `groupadd` adds a group
	- `programmers` name of the group
	- `-comment "Dev Team"` adds a comment for future understanding
- To create a User: `pveum useradd devuser@pve -groups programmers`
	- Here, `useradd` adds a new user
	- `devuser@pve` user name is 'devuser' and is going to be a user in 'pve' hostname.
	- `-groups programmers` add this new user in 'programmers' group
- to set password of a user: `passwd devuser`
**Ubuntu**:
- Create a group: `groupadd nasgroup`
- To create a User: `useradd -m -g nasgroup newnasuser`
	- `-m` create a home directory for the new user
	- `-g nasgroup` add this new user in 'nasgroup' group
	- `newnasuser` name of the user
- if we want to create a user but not let the user have a home directory or use shell, something like `systemd`, we can make them using: `useradd -M -s /usr/sbin/nologin nasuser`
	- `-M`: no home directory will be created
	- `-s /usr/sbin/nologin` This user cannot use interactable shell
- to set password of a user: `passwd newnasuser`

#### low level (with specific ids)
**Proxmox**:
- To check if the GID is already taken: `getent group 2000`
	- If this returns nothing, the ID is unused and we can use it for new groups.
	- If the id passed is in use (e.g., 997), we get data about the user group using this id: `input:x:997:`
- Create a group: `groupadd -g 2000 mycustomgroup`
	- same as linux, we are using `groupadd` to add a group
	- `-g 2000`: This flag sets the Group ID (GID) to exactly 2000.
	- `mycustomgroup`: name of the group
- Check if the UID is already taken `getent passwd 1005`
	- If this returns nothing, the ID is unused and we can use it for new users.
	- If the id passed is in use (e.g., 3), we get data about the user using this id: `sys:x:3:3:sys:/dev:/usr/sbin/nologin`
- Create a user: `useradd -u 1005 -m -s /bin/bash customuser`
	- Here, instead of `pveum` (Proxmox VE User Manager) we are using `useradd` which is a standard Ubuntu low level command so we can have better control on the passed variables and the options used to define this user.
	- `-u 1005` creating a user with UID 1005
	- `-m` create a home directory for this user
	- `-s /bin/bash` we are using `/bin/bash` shell, so user can use SSH and login and use command line, it is not a system user, but a real usre.
	- `customuser` name of the user

> [!Warning]
> Even though new users and groups are added to PAM(Pluggable Authentication Modules) in the underlying Ubuntu architecture, our Proxmox OS is still unaware of these and needs additional instructions to add these users and groups to Proxmox VE Authentication Server (@pve).

**Ubuntu**:
- To create a User group: `groupadd -g 1010 nas-data-group`
	- Here, `groupadd` adds a new group with group id (GID) as '1010' and name as 'nas-data-group'
- Create a specific User ID assigned to that Group: `useradd -u 1010 -g 1010 -m -s /bin/bash nasadmin`
	- create a new user with ID '1010' and add this new user to primary group of '1010'('nas-data-group' created above).
	- the new user will get a Home directory
	- the new user can login to `bin/bash` (interactable shell)
	- the new user will be named `nasadmin`

Summary Comparison

|Feature|Linux PAM (`@pam`)|Proxmox VE (`@pve`)|
|---|---|---|
|**Storage**|`/etc/passwd` & `/etc/shadow`|`/etc/pve/priv/shadow.cfg`|
|**Creation**|Via Command Line (`useradd`)|Via Proxmox GUI or `pveum`|
|**Shell/SSH**|Yes (standard Linux behavior)|No (GUI/API only)|
|**Clustering**|Local to one node|Synchronized across cluster|

To change a PAM user to PVE user:
- let's say the PAM user is `useradd -u 1005 -m -s /bin/bash customuser` (Now the OS knows who they are).
- Add to PVE: `pveum useradd customuser@pam` (Now the Proxmox GUI knows they exist).
- Assign Roles: `pveum acl modify /vms/100 -user customuser@pam -role PVEVMAdmin` (Now they can actually do things).


```sh
# 1. Create the Group with GID 2000
groupadd -g 2000 devs

# 2. Create the User with UID 2000 and primary group 'devs'
useradd -u 2000 -g 2000 -m -s /bin/bash username01

# 3. Tell Proxmox GUI about the user (using the PAM realm)
pveum useradd username01@pam

# 4. (Optional) Create a Proxmox-level group to manage GUI permissions
pveum groupadd PVE_DEVS -comment "Developers Group"
pveum useradd username01@pam -groups PVE_DEVS
```

### Find details about user or group
**Proxmox**:
CLI:
```sh
root@pve:~# # List all PVE Users
root@pve:~# pveum user list
┌──────────┬─────────┬──────────────────────────┬────────┬────────┬───────────┬────────┬──────┬──────────┬────────────┬──────────────────┬────────┬─────────────┐
│ userid   │ comment │ email                    │ enable │ expire │ firstname │ groups │ keys │ lastname │ realm-type │ tfa-locked-until │ tokens │ totp-locked │
╞══════════╪═════════╪══════════════════════════╪════════╪════════╪═══════════╪════════╪══════╪══════════╪════════════╪══════════════════╪════════╪═════════════╡
│ root@pam │         │ thegamelearner@gmail.com │ 1      │      0 │           │        │      │          │ pam        │                  │        │             │
└──────────┴─────────┴──────────────────────────┴────────┴────────┴───────────┴────────┴──────┴──────────┴────────────┴──────────────────┴────────┴─────────────┘
root@pve:~# # List all PVE groups
root@pve:~# pveum group list
root@pve:~# # List all PVE Users in pretty JSON format
root@pve:~# pvesh get /access/users --output-format=json-pretty
[
   {
      "email" : "thegamelearner@gmail.com",
      "enable" : 1,
      "expire" : 0,
      "realm-type" : "pam",
      "userid" : "root@pam"
   }
]
root@pve:~# # List all PVE User groups in pretty JSON format
root@pve:~# pvesh get /access/groups --output-format=json-pretty
[]
root@pve:~# # List all underlaying Ubuntu Users
root@pve:~# getent passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
_chrony:x:100:102:Chrony daemon:/var/lib/chrony:/usr/sbin/nologin
dhcpcd:x:101:65534:DHCP Client Daemon:/usr/lib/dhcpcd:/bin/false
messagebus:x:992:992:System Message Bus:/nonexistent:/usr/sbin/nologin
tcpdump:x:102:103::/nonexistent:/usr/sbin/nologin
sshd:x:990:65534:sshd user:/run/sshd:/usr/sbin/nologin
_rpc:x:103:65534::/run/rpcbind:/usr/sbin/nologin
statd:x:104:65534::/var/lib/nfs:/usr/sbin/nologin
postfix:x:105:104:Postfix MTA:/var/spool/postfix:/usr/sbin/nologin
frr:x:106:107:Frr routing suite:/nonexistent:/usr/sbin/nologin
tss:x:107:109:TPM software stack:/var/lib/tpm:/bin/false
ceph:x:64045:64045:Ceph storage service:/var/lib/ceph:/usr/sbin/nologin
root@pve:~# # Get user id of root user in underlying Ubuntu machine
root@pve:~# id -u root
0
root@pve:~# 
```
GUI:
![021 Proxmox users.png](/img/user/All%20Published%20Notes/Homelab/Images/021%20Proxmox%20users.png)

**Ubuntu**:
```sh
root@NAS-LXC:~# # find user's id (UID) using username
root@NAS-LXC:~# id -u nasuser
1000
root@NAS-LXC:~# # find group's id (GID) using groupname
root@NAS-LXC:~# id nasuser
uid=1000(nasuser) gid=1000(nasuser) groups=1000(nasuser)
root@NAS-LXC:~# # find group's details using id (GID)
root@NAS-LXC:~# id 1000   
uid=1000(nasuser) gid=1000(nasuser) groups=1000(nasuser)
root@NAS-LXC:~# # find user's group details using username
root@NAS-LXC:~# groups nasuser
nasuser : nasuser
root@NAS-LXC:~# # get user details using user id (UID)
root@NAS-LXC:~# getent passwd 1000
nasuser:x:1000:1000::/home/nasuser:/usr/sbin/nologin
root@NAS-LXC:~#
```

### Change User group
**Proxmox**:
Proxmox allows PVE user addition, editing, group assigning and removal in web GUI and the options are simple and exploratory.
For PAM users, the code is same as Ubuntu's CLI commands, so you can read them below.
**Ubuntu**:
list existing groups of a user: `groups username` or `id username`
remove existing user(username) from an existing group(groupname): `sudo gpasswd -d username groupname`
Add existing user to a new existing group while retaining all previously assigned groups: `sudo usermod -aG groupname username`
- `-a` : appends user to the group without removing them from their current groups.
- `-G`: Tells the system you are defining supplementary groups.

### Summary
Proxmox uses Linux but has more on top of it than simple UI. The user management of Linux can be used in Proxmox to create PAM user, but these users cannot log in to Proxmox to do activities. The Proxmox users are PVE users who can do a lot more and the main user of the OS.

When needed, specially for privileged containers, the user access of Proxmox matches the one in the container, so try to maintain distinct user id when generating a new user id in any container and map it to same PAM user id in Proxmox with necessary permissions. 

If Proxmox user does not exists, or does not have necessary permissions, chances are your container's user will also not have necessary permissions even if you give it necessary permissions in the container.

> [!Note]
> If you are **using the PVE** realm, you **cannot set a custom UID or GID**. The IDs are internal to Proxmox and are not used for file ownership on the host.










---

[^1]:
[^2]:

