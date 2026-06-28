---
{"dg-publish":true,"permalink":"/all-published-notes/software/computer/personal-changes/shell-scripts/manually-loaded/system/list-extensions/"}
---

created: 2026-01-01
updated: 2026-01-01
### list_extensions
Lists all file extensions present in the folder

### code
Usage:

| Command           | use case                           |
| ----------------- | ---------------------------------- |
| `list_extensions` | gives all extensions in the folder |

Code:
```sh title:list_extensions
find . -type f | sed -E 's/.*\.//' | sed '/\//d' | sort -u
```

adding this to bin:

```bash title:Usage, hl:1,8,5
thegamelearner@thegamelearner-MS-7E12:~/Music$ ./list_extensions 
flac
mp3
xspf
thegamelearner@thegamelearner-MS-7E12:~/Music$ mv list_extensions ~/bin/list_extensions
thegamelearner@thegamelearner-MS-7E12:~/Music$ ls
 addMp3Extension   AllSongs.xspf  'Music Library'   UnTagged
thegamelearner@thegamelearner-MS-7E12:~/Music$ list_extensions 
flac
mp3
xspf
thegamelearner@thegamelearner-MS-7E12:~/Music$ 
```

---


[^1] [[All Published Notes/Software/Computer/Personal Changes/Shell Scripts/manually-loaded/System/Adding personal commands to system\|Adding personal commands to system]]