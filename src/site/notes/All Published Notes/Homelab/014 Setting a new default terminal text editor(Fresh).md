---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/014-setting-a-new-default-terminal-text-editor-fresh/"}
---

created: 2026-06-11
updated: 2026-06-11

Mostly when we are editing file like git merge conflict or system config, the terminal has it's default editor which many suggest to change and more than many have said to use 'nano'.

I am a lazy programmer, and setting up a terminal with new shortcuts that I have to remember is not something I love.

By default, we only get limited choices for file editors in the terminal, for this I always use 'Fresh' as I loved the ease of file editing given by it.

To confirm the default choice, I will use the command `crontab -e` which opens a file in the default editor without giving a choice for which editor to use, I will then change default editor to fresh editor and test again.

```sh
root@DockerHost:~# # command to open default editor, this is useful for automated tasks in terminal, so make sure to not edit it
root@DockerHost:~# crontab -e
no crontab for root - using an empty one
root@DockerHost:~# 
root@DockerHost:~# # Confirm the path of the editor we want to use
root@DockerHost:~# which fresh
/bin/fresh
root@DockerHost:~# select-editor

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.tiny
  3. /bin/ed

Choose 1-3 [1]: 1
root@DockerHost:~# # try to export your choice ('fresh) in bashrc
root@DockerHost:~# echo 'export EDITOR="/bin/fresh"' >> ~/.bashrc && source ~/.bashrc
root@DockerHost:~# # updating bashrc did not work for me.
root@DockerHost:~# # remove the older options so we can force our choice as the only choice in selected_editor.
root@DockerHost:~# rm -f ~/.selected_editor
root@DockerHost:~# echo 'export VISUAL="/bin/fresh"' >> ~/.bashrc
root@DockerHost:~# source ~/.bashrc
root@DockerHost:~# # now the 'crontab -e' command will open the file in fresh editor.
root@DockerHost:~# crontab -e
no crontab for root - using an empty one
crontab: installing new crontab
root@DockerHost:~# # after saving the file changes, we will return here
root@DockerHost:~# 
```

Now, my system always opens all files in fresh("/bin/fresh") instead of giving me outdated choices.




---

[^1]:
[^2]:

