---
{"dg-publish":true,"permalink":"/all-published-notes/computer/os/chrome-os/creating-ssh-server-in-chromebook/"}
---

# Creating SSH server in Chromebook
posted: 2025-07-26

### Defining the username and password:
- We need a account password, if it was not set, the authentication from the machines can fail
- \[ChromeOS\] \[Terminal\] Change to superuser, as it can set other user's password: `sudo su`
- \[ChromeOs\] \[Terminal\] then enter `passwd` to change the root password, or use `passwd thegamelearner` to set password for the user 'thegamelearner', after setting the password, you can exit or stay here as a root user
- \[ChromeOs\] \[Terminal\] Let us assume the password is : 'GameLearning$#22'

### Check the status of openssh-server
- \[ChromeOs\] \[Terminal\] check the current system status by starting the server: `sudo service ssh start`
	- If this fails because of `conditionPathExists=~/etc/ssh/sshd_not_to_be_run was not met`, we can either delete this file, ==or==
	- we can purge the 'openssh-server' and then install it again: `sudo apt purge openssh-server && sudo apt install openssh-server`
- \[ChromeOs\] \[Terminal\] stop the server if it was started, as default options use port 22, which is not a great option: `sudo service ssh stop`

### Port Forwarding on desired port
- \[ChromeOs\] \[Settings\] open settings, in linux port forwarding, add/enable the port(between 1024 and 65535) you want to use for ssh
- \[ChromeOs\] \[Settings\] I am keeping both TCP and UDP active as I am not sure which to use, in my case port is: '1234'
- \[ChromeOs\] \[Terminal\] Modify the SSH config file at `/etc/ssh/sshd_config` using something like vim or nano, I used micro with sudo permission: `sudo micro /etc/ssh/sshd_config`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] add the following lines:
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `Port 1234`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `AllowUsers itrishabhjain`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `PermitRootLogin no`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `ListenAddress 0.0.0.0`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `PasswordAuthentication yes`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `AllowAgentForwarding yes`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] `AllowTcpForwarding yes`
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] save the file and exit : 'ctrl+s' and then 'ctrl+q'
- \[ChromeOs\] \[Terminal\] \[/etc/ssh/sshd_config\] only add the texts inside the quote, eg., for `Port 1234`, type 'Port 1234' in a single line without the quotes
- \[ChromeOs\] \[Terminal\] verify the edited file is correct in syntax : `sudo sshd -t`
- \[ChromeOs\] \[Terminal\] the new port needs to be set in '/etc/systemd/system/ssh.socket.d/override.conf' as well, because sometimes ChromeOs does not allow us to use new port for ssh
- \[ChromeOs\] \[Terminal\] edit the socket file by `sudo systemctl edit ssh.socket` which will edit '/etc/systemd/system/ssh.socket.d/override.conf' in the default editor
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] add the following lines:
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] `[Socket]`
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] `ListenStream=`
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] `ListenStream=1234`
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] save the file and exit : 'ctrl+s' and then 'ctrl+q'
- \[ChromeOs\] \[Terminal\] \[/etc/systemd/system/ssh.socket.d/override.conf\] 'ListenStream=' is important as it removes any old port being used.
- \[ChromeOs\] \[Terminal\] reload the systemd daemon : `sudo systemctl daemon-reload` which will use updated configurations meaning the new port you defined.

---
### start ssh service
- \[ChromeOs\] \[Terminal\] Restart the SSH Services: `sudo systemctl restart ssh.socket` and `sudo systemctl restart ssh.service` or you can use `sudo service ssh start`
- \[ChromeOs\] \[Terminal\] verify the port has changed: `sudo service ssh status` should return the new port in use.
- \[ChromeOs\] open wifi setting and get the ip address of the system, click on wifi, go into wifi connected, use the 'i' button to see ip address, e.g., '192.168.31.48'. Most of the times, it will start with '192.168.'

---
### Connect to the created SSH
- \[linux\] \[Terminal\] open terminal in linux machine, use the ssh command to tunnel into chromeOS `ssh -p 1234 itrishabhjain@192.168.31.48`
- \[linux\] \[Terminal\] if asked 'The authenticity of host '\[192.168.31.48\]:1234 (\[192.168.31.48\]:1234)' can't be established.', 'Are you sure you want to continue connecting (yes/no/\[fingerprint\])?' type `yes`
- \[linux\] \[Terminal\] prompted for password, type the password set in ChromeOS

# Footnotes
[^1]:
[^2]:

