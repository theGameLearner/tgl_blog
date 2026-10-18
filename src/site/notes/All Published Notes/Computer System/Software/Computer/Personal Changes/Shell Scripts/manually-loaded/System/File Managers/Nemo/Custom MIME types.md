---
{"dg-publish":true,"permalink":"/all-published-notes/computer-system/software/computer/personal-changes/shell-scripts/manually-loaded/system/file-managers/nemo/custom-mime-types/"}
---

created: 2026-10-18
updated: 2026-10-18

### Defining the Action
We have a nice addition in nemo browser that allows us to show text files as "Text" instead of `.txt` in types, but the problem with these is that it also shows `.meta` files as "Text".
"Nemo" allows us to show custom MIME values in "Types" column, which we can update or edit if a option does not suit our taste.
I will update it so that `.meta` files appear as "Unity Meta File", as I have not seen this extension in any other files that "I" use.

### Action
To do this, we create a file `custom-code-types.xml`, which is stored in `~/.local/share/mime/packages` of the user.
steps:
- make MIME Definition File
- Make the file using 'xed' or 'code'(if you  have visual studio code)
	- update file content, if you already have the file.
	- define 'mime-type'
		- a general problem is Nemo uses the parent types sometimes. So in such cases, we can use : `<mime-type type="application/x-unity-meta">` or `<mime-type type="application/x-csharp">` instead of `<mime-type type="text/x-unity-meta">` which is under text(`<sub-class-of type="text/plain"/>`).
	- `<comment>`: A human-readable description.
	- `<glob pattern="*.ext"/>`: This associates the MIME type with the file extension.
- Update the MIME database
- do a system-wide update (though local changes often override)
- Restart Nemo

#### Commands
create the MIME `xml` file:
```bash
mkdir -p ~/.local/share/mime/packages
cd ~/.local/share/mime/packages
xed custom-mime-types.xml
```

Paste this template
```xml
<?xml version="1.0" encoding="UTF-8"?>
<mime-info xmlns="http://www.freedesktop.org/standards/shared-mime-info">
    <!-- MIMEs for Unity .meta files -->
    <mime-type type="application/x-unity-meta">
        <comment>Unity Meta File</comment>
        <glob pattern="*.meta" weight="80"/>  <!-- Higher priority than system defaults -->
        <magic priority="60">  <!-- Content-based detection -->
            <!-- match tag defines the alternative matching conditions for the file to be tagged -->
            <match type="string" value="fileFormatVersion:" offset="0"/>
            <!-- check if the content of the file starts with(offset="0") the string "fileFormatVersion:" -->
        </magic>
    </mime-type>
    
    <!-- MIME for C# files -->
    <mime-type type="application/x-csharp">
        <comment>C# Source Code</comment>
        <glob pattern="*.cs" weight="80"/>
        <magic priority="60">
            <!-- match tag defines the alternative matching conditions for the file to be tagged -->
            <!-- if a file starts with 'using' or 'namespace', we assume it to be a csharp file -->
            <match type="string" value="using " offset="0"/>
            <match type="string" value="namespace " offset="0"/>
        </magic>
    </mime-type> 
    
    <!-- MIME for .gitignore files -->
    <mime-type type="application/x-csharp">
        <comment>Git Ignore</comment>
        <glob pattern=".gitignore" weight="80"/>
    </mime-type>
</mime-info>
```

Remove any old database (*if it exists*):
```bash
rm -f ~/.local/share/mime/mime.cache
```

Update the database:
```bash
update-mime-database ~/.local/share/mime
```

sudo update the database if you need it to be global(*Optional*):
```bash
sudo update-mime-database /usr/share/mime
```

check for errors:
```
echo $?
```

Reset the Nemo:
```bash
# Completely quit Nemo
nemo -q

# Kill lingering processes
pkill -f nemo-desktop
pkill -f nemo

# Clear Nemo's cache
rm -rf ~/.cache/nemo/*
rm -rf ~/.local/share/nemo/actions

# Restart Nemo after delay
sleep 2
nemo &
```

Check a meta file that should be recognized:
```bash
xdg-mime query filetype ~/path/to/file/README.md.meta
```

*If Nemo doesn't show the updated change*:
create a MIME file to override the association:
```bash
mkdir -p ~/.config/mimeapps.list
echo -e '[Added Associations]\napplication/x-csharp=code.desktop\napplication/x-unity-meta=code.desktop' > ~/.config/mimeapps.list
```
This associates a program that opens a file with a software which can open it.


### Issues / Limitations
#### File types defined in mime package
I tried to add types as "Audio/Mp3" or "Audio_Mp3" for `.mp3` files, but it failed.
I have confirmed with Gemini AI and the reason is:
> The reason system update steps haven't worked is that the shared-mime-info database treats existing system MIME definitions (like `audio/mpeg` or `audio/flac`) as already defined in `/usr/share/mime/packages/freedesktop.org.xml`. When you redefine an existing type name in user space, `update-mime-database` merges comments, and GLib/Nemo continues resolving the primary system description or general parent media category ("Audio").

So, this option works for custom types, but will not work well if the files are already specified in `/usr/share/mime/packages/freedesktop.org.xml` (about 45K lines of data in this file). I can see that this file has the type defined for all languages, so we cannot edit it safely
```sh
Sun Oct 18, 11:15:27 PM "~/Documents"|
thegamelearner@thegamelearner-MS-7E12:$ cat -n /usr/share/mime/packages/freedesktop.org.xml | grep .mp3
 25863      <alias type="audio/x-mp3"/>
 25866      <alias type="audio/mp3"/>
 25876      <glob pattern="*.mp3"/>
 25906      <alias type="audio/x-mp3-playlist"/>


Sun Oct 18, 11:17:11 PM "~/Documents"|
thegamelearner@thegamelearner-MS-7E12:$ cat -n /usr/share/mime/packages/freedesktop.org.xml | sed -n '25800,25900p'

 25800      <comment xml:lang="ar">صوت MP2</comment>
 25801      <comment xml:lang="af">MP2-oudio</comment>
 25802      <alias type="audio/x-mp2"/>
 25803      <glob pattern="*.mp2"/>
 25804    </mime-type>
 25805    <mime-type type="audio/mpeg">
 25806      <comment>MP3 audio</comment>
 25807      <comment xml:lang="zh_TW">MP3 音訊</comment>
 25808      <comment xml:lang="zh_CN">MP3 音频</comment>
 25809      <comment xml:lang="vi">Âm thanh MP3</comment>
```


---

[^1]: 
[^2]: 

