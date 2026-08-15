---
{"dg-publish":true,"permalink":"/all-published-notes/computer-system/software/computer/development/markdown-generator/python/markdown-file-generator-python/"}
---

created: 2026-07-27
updated: 2026-07-27

### Template
Let us move to a new folder where I will create and store the template file(`HowToAddPackage.md.tmpl`) which will be used as a reference for creating the target markdown file.

```sh
Mon Jul 27, 10:42:05 PM "~/Documents/Unity Projects/All_Plugins/Assets/All Modules/TutorialHandler"|
thegamelearner@thegamelearner-MS-7E12:$ cd "/home/thegamelearner/Documents/Learning/pythonLearning"

Mon Jul 27, 10:42:15 PM "~/Documents/Learning/pythonLearning"|
thegamelearner@thegamelearner-MS-7E12:$ ls -al
total 12
drwxrwxr-x 2 thegamelearner thegamelearner 4096 Jul 27 22:41 .
drwxrwxr-x 3 thegamelearner thegamelearner 4096 Jul 27 22:41 ..
-rw-rw-r-- 1 thegamelearner thegamelearner 3536 Jul 27 21:24 HowToAddPackage.md.tmpl

Mon Jul 27, 10:42:19 PM "~/Documents/Learning/pythonLearning"|
thegamelearner@thegamelearner-MS-7E12:$ 
```

In this file, every argument I can use is written in `${variable_name}` format:
- *${git_url}*: 'https://github.com/tglGames-Plugins/tgl-tutorial-manager.git'
- *${display_name}*: 'TGL Tutorial Manager'
- *${project_root}*: `/home/thegamelearner/Documents/Unity Projects/TestTutorial/`
- *${toml_file_path}*: `/home/thegamelearner/Documents/Unity Projects/TestTutorial/Assets/AccessConfig/upmconfig.toml`
- *${unity_editor_path}*: `/home/thegamelearner/Unity/Hub/Editor/6000.2.6f2/Editor/Unity`
- *${package_url}*: `com.tglgames.tgl-tutorial-manager`
- *${version}*: '1.0.2'

> [!Warning]
> Make sure not to leave any line with `$` symbol as python will try to create variable and fail there. use `$$` in the template to generate `$` in the generated file.

### python script
We will now create a GUI version using Python's built-in `tkinter` library.

Python GUI Script (`gui_generator.py`):
```python
import os
import tkinter as tk
from tkinter import filedialog, messagebox, ttk
from string import Template

class MarkdownGeneratorGUI:
    def __init__(self, root):
        self.root = root
        self.root.title("Unity Documentation Generator")
        self.root.geometry("680x520")

        # Set minimum window dimensions and allow resizing
        self.root.minsize(680, 520)
        self.root.resizable(True, True)

        # Main frame padding
        main_frame = ttk.Frame(root, padding="15")
        main_frame.pack(fill=tk.BOTH, expand=True)

        # Configure column weight so entry fields expand nicely when resized horizontally
        main_frame.columnconfigure(1, weight=1)

        # Data model for input fields (git_url, display_name, package_url start empty)
        self.fields = {
            "git_url": ("Git Repository URL:", ""),
            "display_name": ("Package Display Name:", ""),
            "project_root": ("Project Root Path:", "/home/thegamelearner/Documents/Unity Projects/TestTutorial/"),
            "toml_file_path": ("TOML File Path:", "/home/thegamelearner/Documents/Unity Projects/TestTutorial/Assets/AccessConfig/upmconfig.toml"),
            "unity_editor_path": ("Unity Editor Path:", "/home/thegamelearner/Unity/Hub/Editor/6000.2.6f2/Editor/Unity"),
            "package_url": ("Scoped Package Name:", ""),
            "version": ("Package Version:", "1.0.2")
        }

        self.entries = {}

        # Render Form Inputs
        row = 0
        for key, (label_text, default_val) in self.fields.items():
            ttk.Label(main_frame, text=label_text).grid(row=row, column=0, sticky=tk.W, pady=4)

            entry = ttk.Entry(main_frame, width=50)
            if default_val:
                entry.insert(0, default_val)
            entry.grid(row=row, column=1, sticky=tk.EW, padx=(5, 5), pady=4)
            self.entries[key] = entry

            # Add path browse buttons for path fields
            if "path" in key or "root" in key:
                btn = ttk.Button(main_frame, text="Browse...", command=lambda e=entry, k=key: self.browse_path(e, k))
                btn.grid(row=row, column=2, pady=4)

            row += 1

        ttk.Separator(main_frame, orient='horizontal').grid(row=row, columnspan=3, sticky="ew", pady=15)
        row += 1

        # Save Button
        self.generate_btn = ttk.Button(
            main_frame,
            text="Generate Markdown File",
            command=self.generate_file
        )
        self.generate_btn.grid(row=row, columnspan=3, ipady=5, sticky="ew")

    def browse_path(self, entry_widget, key):
        if "root" in key:
            selected_path = filedialog.askdirectory(title="Select Project Root Directory")
        else:
            selected_path = filedialog.askopenfilename(title="Select File Path")

        if selected_path:
            entry_widget.delete(0, tk.END)
            entry_widget.insert(0, selected_path)

    def generate_file(self):
        # 1. Locate template
        template_file = "HowToAddPackage.md.tmpl"
        if not os.path.exists(template_file):
            messagebox.showerror(
                "Error",
                f"Template file '{template_file}' not found in current directory!"
            )
            return

        # 2. Collect field values
        values = {key: entry.get().strip() for key, entry in self.entries.items()}

        # 3. Validate required fields
        required_fields = {
            "git_url": "Git Repository URL",
            "display_name": "Package Display Name",
            "project_root": "Project Root Path:",
            "toml_file_path": "TOML File Path:",
            "unity_editor_path": "Unity Editor Path:",
            "package_url": "Scoped Package Name",
            "version": "Package Version:"
        }

        missing_fields = [label for key, label in required_fields.items() if not values.get(key)]

        if missing_fields:
            messagebox.showwarning(
                "Missing Information",
                f"Please fill in the following required field(s):\n- " + "\n- ".join(missing_fields)
            )
            return

        # 4. Prompt user for output save location
        save_path = filedialog.asksaveasfilename(
            title="Save Markdown File As...",
            defaultextension=".md",
            initialfile="HowToAddPackage.md",
            filetypes=[("Markdown files", "*.md"), ("All files", "*.*")]
        )

        if not save_path:
            return  # User cancelled dialog

        # 5. Substitute & Write
        try:
            with open(template_file, "r", encoding="utf-8") as tf:
                template = Template(tf.read())

            output_content = template.substitute(values)

            with open(save_path, "w", encoding="utf-8") as of:
                of.write(output_content)

            messagebox.showinfo("Success", f"File created successfully at:\n{save_path}")
        except Exception as e:
            messagebox.showerror("Error", f"An error occurred while writing:\n{str(e)}")


if __name__ == "__main__":
    root = tk.Tk()
    app = MarkdownGeneratorGUI(root)
    root.mainloop()
       
```

### Install dependencies
In some versions python is pre-installed but not `tkinter`, if you get errors due to it, install it:
```sh
sudo apt install python3-tk
```

### how to use
To use this script, run `python3 generate_docs.py` on same folder:
```sh

Mon Jul 27, 11:01:32 PM "~/Documents/Learning/pythonLearning"|
thegamelearner@thegamelearner-MS-7E12:$ ls -al
total 20
drwxrwxr-x 2 thegamelearner thegamelearner 4096 Jul 27 22:53 .
drwxrwxr-x 3 thegamelearner thegamelearner 4096 Jul 27 22:41 ..
-rw-rw-r-- 1 thegamelearner thegamelearner 4218 Jul 27 22:57 gui_generator.py
-rw-rw-r-- 1 thegamelearner thegamelearner 3141 Jul 27 22:55 HowToAddPackage.md.tmpl

Mon Jul 27, 11:01:36 PM "~/Documents/Learning/pythonLearning"|
thegamelearner@thegamelearner-MS-7E12:$ python3 gui_generator.py 
```

This opens the dialogue box with data to be entered.

![python tkinter default UI.png](/img/user/All%20Published%20Notes/Computer%20System/Software/Computer/Development/Markdown%20Generator/images/python%20tkinter%20default%20UI.png)

#### Updating UI (Optional)
As the UI looks too old, I will update it to look similar to OS based UI:
```python
class MarkdownGeneratorGUI:
    def __init__(self, root):
        self.root = root
        self.root.title("Unity Documentation Generator")
        self.root.geometry("680x520")
        self.root.minsize(680, 520)

        # 1. Initialize ttk Style
        style = ttk.Style()
        
        # 2. Pick the native theme available on the system
        available_themes = style.theme_names()
        if "clam" in available_themes:
            style.theme_use("clam")  # Looks much cleaner and native on Linux
        elif "vista" in available_themes:
            style.theme_use("vista") # Native look on Windows
```

The available themes are:
- Linux
	- `clam`
	- `alt`
	- `default`
- Windows
	- `vista`
	- `xpnative`
- macOS
	- `aqua`
The results are quite bad.


I can install and use `customtkinter` for updated UI if I want to change it later:
```sh
pip install customtkinter
```
But this is a helper, so I do not want to spend too much time for it.












---

[^1]: 
[^2]: 

