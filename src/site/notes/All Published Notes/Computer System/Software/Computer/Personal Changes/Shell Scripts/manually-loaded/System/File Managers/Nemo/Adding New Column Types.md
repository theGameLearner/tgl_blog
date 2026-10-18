---
{"dg-publish":true,"permalink":"/all-published-notes/computer-system/software/computer/personal-changes/shell-scripts/manually-loaded/system/file-managers/nemo/adding-new-column-types/"}
---

created: 2026-10-18
updated: 2026-10-19

### Defining the Action
I am using "Nemo" File Manager and also have a directory with a lot of music files, the columns in list view defines them as "Audio" but does not mention whether it is MP3 or FLAC or WAV, this can be an issue for organization or when I am making a playlist.

I decided to update my nemo file browser to show a new column called "Extension" which will show all extensions, I am not sure why, but this column option is not available in nemo.

![Nemo File List Columns.png](/img/user/All%20Published%20Notes/Computer%20System/Software/Computer/Personal%20Changes/Shell%20Scripts/manually-loaded/System/File%20Managers/Nemo/Images/Nemo%20File%20List%20Columns.png)

I used github[^1] to find a user who faced this issues and made an extension, and edited it so that I do not take all extensions but only ones that serve my purpose.
If you need this extension, we can install it with ```
```
sudo apt install nemo-media-columns
```
But, as I need to differentiate the extensions and not list Artists or other Metadata, I will create a new column from scratch.

This is different than adding a custom MIME types, which I have done to show Unity `.meta` files as "Unity Meta Files" in [[All Published Notes/Computer System/Software/Computer/Personal Changes/Shell Scripts/manually-loaded/System/File Managers/Nemo/Custom MIME types\|Custom MIME types]].

### Action

#### Add All Prerequisites

We need the `nemo-python` library to edit the nemo package, so first install it:
```sh
sudo apt install nemo-python
```

After the package is installed, we need a custom directory where all of nemo extensions are stored: `~/.local/share/nemo-python/extensions`. If the directory was not created before, create it now:
```sh
mkdir -p ~/.local/share/nemo-python/extensions
```

#### Create the "Extension" Column file

Lets create a new file `nemo-extension-column.py` in `~/.local/share/nemo-python/extensions` directory
```sh
zed ~/.local/share/nemo-python/extensions/nemo-extension-column.py
```

`nemo-extension-column.py` File:
```py
#!/usr/bin/python3
"""
Nemo extension: adds an "Extension" column to the list view.

- Files with an extension    → extension in lowercase, without the dot (e.g. "jpg")
- Files without an extension → "file"
- Directories                → "Folder"
- Non-local URIs (network shares, etc.) → column left blank
"""

import os
from urllib.parse import unquote, urlparse

import gi
gi.require_version('Nemo', '3.0')
from gi.repository import GObject, Nemo


class ExtensionColumnExtension(GObject.GObject, Nemo.ColumnProvider, Nemo.InfoProvider):
    def __init__(self):
        super().__init__()

    def get_columns(self):
        return (
            Nemo.Column(
                name="NemoPython::file_extension_column",
                attribute="file_extension",
                label="Extension",
                description="Displays file extension (lowercase, without leading dot)"
            ),
        )

    def update_file_info(self, file):
        scheme = file.get_uri_scheme()

        # Handle local files as well as shortcuts in Recent/Favorites
        if scheme == 'file':
            uri = file.get_uri()
        elif scheme in ('recent', 'favorites'):
            uri = file.get_activation_uri()
        else:
            return

        if not uri:
            return

        parsed = urlparse(uri)
        if parsed.scheme != 'file':
            return

        # Check directory status via Nemo's cached metadata (avoids extra disk stats)
        if file.is_directory():
            file.add_string_attribute('file_extension', 'Folder')
            return

        # Extract file name safely from path or fallback to URL-decoded path
        filename = file.get_name() if scheme == 'file' else unquote(parsed.path)

        # Extract and format extension
        _, ext = os.path.splitext(filename)
        if ext:
            file.add_string_attribute('file_extension', ext[1:].lower())
        else:
            file.add_string_attribute('file_extension', 'file')

```


After the file is saved, quit all nemo sessions and restart nemo:
```sh
nemo -q
```

Now we should be able to find the newly added "Extension" column in list view

#### (Optional) Make the Column Default for All Folders

If you want the "Extension" column to appear automatically in every folder:
- Go to `Edit` → `Preferences` → `List Columns`.
- Move "Extension" to the "Visible Columns" list.
- This setting will be applied to all future folder views.







---

[^1]: https://github.com/linuxmint/nemo-extensions/blob/master/nemo-media-columns/nemo-media-columns.py
[^2]: 

