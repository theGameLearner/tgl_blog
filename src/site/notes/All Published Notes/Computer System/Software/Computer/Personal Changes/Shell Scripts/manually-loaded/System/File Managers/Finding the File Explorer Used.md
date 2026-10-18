---
{"dg-publish":true,"permalink":"/all-published-notes/computer-system/software/computer/personal-changes/shell-scripts/manually-loaded/system/file-managers/finding-the-file-explorer-used/"}
---

created: 2026-10-18
updated: 2026-10-18

Linux has so many options that knowing your OS is necessary to do any activity

Depending on your scenario, there are easy and tough ways to identify the file explorer you use:

*For someone who does not change things too often*:
We can find the file explorers installed using `which` command followed by all commonly used file explorers.

```sh
Sun Oct 18, 09:53:32 PM "~/Documents"|
thegamelearner@thegamelearner-MS-7E12:$ which nautilus dolphin thunar pcmanfm nemo caja doublecmd mc xfe 2>/dev/null
/usr/bin/dolphin
/usr/bin/nemo

Sun Oct 18, 10:26:47 PM "~/Documents"|
thegamelearner@thegamelearner-MS-7E12:$ 
```

*For multiple file explorer users or users who might install and uninstall them*:

You can run multiple commands to find all file explorers installed:
```sh
# From Applications(`/usr/share/applications`) find all apps that are in FileManager category
Sun Oct 18, 10:29:50 PM "~/Documents"|
thegamelearner@thegamelearner-MS-7E12:$ grep -l "Categories=.*FileManager" /usr/share/applications/*.desktop
/usr/share/applications/org.kde.dolphin.desktop

# From Applications(`/usr/share/applications`) find all apps that are in FileManager category
Sun Oct 18, 10:35:06 PM "~/Documents"|
thegamelearner@thegamelearner-MS-7E12:$ grep -E "Name=|Categories=" /usr/share/applications/*.desktop | grep -B 1 "FileManager"
/usr/share/applications/org.kde.dolphin.desktop:Name=Dolphin
/usr/share/applications/org.kde.dolphin.desktop:Categories=Qt;KDE;System;FileTools;FileManager;

Sun Oct 18, 10:35:16 PM "~/Documents"|
thegamelearner@thegamelearner-MS-7E12:$ dpkg -l | grep -E "nautilus|dolphin|thunar|pcmanfm|nemo|caja|mc\b"
ii  dolphin                                          4:23.08.5-0ubuntu4                               amd64        file manager
ii  gir1.2-nemo-3.0                                  6.4.5+xia                                        amd64        libraries for nemo components - gir bindings
ii  libdolphinvcs5:amd64                             4:23.08.5-0ubuntu4                               amd64        library to show version control in Dolphin
ii  libnautilus-extension4:amd64                     1:46.4-0ubuntu0.2                                amd64        libraries for nautilus components - runtime version
ii  libnemo-extension1:amd64                         6.4.5+xia                                        amd64        libraries for nemo components - runtime version
ii  nautilus-data                                    1:46.4-0ubuntu0.2                                all          data files for nautilus
ii  nemo                                             6.4.5+xia                                        amd64        file manager and graphical shell for Cinnamon
ii  nemo-data                                        6.4.5+xia                                        all          data files for nemo
ii  nemo-dbg                                         6.4.5+xia                                        amd64        file manager and graphical shell for Cinnamon - debugging version
ii  nemo-emblems                                     6.4.0+xia                                        all          Change a folder or file emblem
ii  nemo-fileroller                                  6.4.0+xia                                        amd64        File Roller integration for Nemo
ii  nemo-preview                                     6.4.0+xia                                        amd64        nemo-preview is a quick previewer for nemo
ii  nemo-share                                       6.4.0+xia                                        amd64        Nemo extension to share folder using Samba
ii  python-nemo                                      6.4.0+xia                                        amd64        Python binding for Nemo components

Sun Oct 18, 10:35:24 PM "~/Documents"|
thegamelearner@thegamelearner-MS-7E12:$ 
```


*For someone who explores all options*:
Close all active file explorers, then open "Activity Monitor" or "System Monitor" and loop at all apps on the top after sorting by App id or process id.
you will find your newly opened file browser on top as it was closed and re-opened:
![File Explorer Process sorted.png](/img/user/All%20Published%20Notes/Computer%20System/Software/Computer/Personal%20Changes/Shell%20Scripts/manually-loaded/System/File%20Managers/Images/File%20Explorer%20Process%20sorted.png)





---

[^1]: 
[^2]: 

