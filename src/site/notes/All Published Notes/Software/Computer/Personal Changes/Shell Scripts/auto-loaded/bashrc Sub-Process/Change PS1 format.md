---
{"dg-publish":true,"permalink":"/all-published-notes/software/computer/personal-changes/shell-scripts/auto-loaded/bashrc-sub-process/change-ps-1-format/"}
---

created: 2026-07-05
updated: 2026-07-05

I want to change how PS1 looks, this is how the command line looks to you:
![terminal ps1 sections.png](/img/user/All%20Published%20Notes/Software/Computer/Personal%20Changes/Shell%20Scripts/auto-loaded/bashrc%20Sub-Process/images/terminal%20ps1%20sections.png)

1. The user name
2. the machine name
3. the current directory path
4. the shell command prompt sign

I found a great online tool that let's me see and try different options before committing to one : [bash prompt generator](https://bash-prompt-generator.org/)

We can use this to see how the prompt can look and then use that in our code.
The prompt I made:
```sh
PS1='\n\[\e[35;48;5;231;1;3m\]\d\[\e[0m\],\[\e[35;1m\] \[\e[0;38;5;232;106m\]\T\[\e[0m\] | \[\e[91;1m\]\u\[\e[36m\]@\[\e[38;5;201m\]\h\[\e[0m\]:"\[\e[38;5;83;1m\]\w\[\e[0m\]"\n\$ '
```

Lets add it to `~/.bashrc` so that it will be auto loaded when I start a session, add it to `~/.bashrc` file at the bottom unless you already have a line about editing ps1, then save. 
In terminal 
```sh
source ~/.bashrc
[info] Currently loaded SSH keys:
256 SHA256:1y3WcQ3VM1PCjztudfrt2Mv/pta7TYyPkHMpmpC+WZ4 thegamelearner@gmail.com (ED25519)
3072 SHA256:UTu22gngWC/dmU+gJVDgkFLUvqoeMS4ThWyl0Gt0TrQ it.rishabh.jain@google.com (RSA)

Sun Jul 05, 05:49:30 | thegamelearner@thegamelearner-MS-7E12:"~/Documents/GithubNotes/TglBlog"
$ 
```
Sorry, terminal output does not display colors:
![terminal ps1 changes.png](/img/user/All%20Published%20Notes/Software/Computer/Personal%20Changes/Shell%20Scripts/auto-loaded/bashrc%20Sub-Process/images/terminal%20ps1%20changes.png)

We can do the same to any shell. 



---

[^1]: 
[^2]: 

