---
{"dg-publish":true,"permalink":"/all-published-notes/computer-system/software/computer/development/markdown-generator/markdown-file-generator/"}
---

created: 2026-07-27
updated: 2026-07-27

## General Idea
As I am making packages for Unity, I have to write multiple files repeatedly like the package.json and the file about how to download the package.
As the content is going to be same for all `HowToAddPackage.md` files with limited names being different, I want a tool that can generate the file on run time.
I will create `package.json` and fill in the same details to a tool and it will create the `HowToAddPackage.md` file as needed.

As there are too many bash scripting already covered, this time I will use other languages to achieve my goal.
- python
	- We can use python + `tkinter` Library
		- [[All Published Notes/Computer System/Software/Computer/Development/Markdown Generator/Python/markdown file generator - python\|markdown file generator - python]]
- Go
	- we can use 'Go + Fyne': Recommended for speed & simplicity
- Rust
	- we can also use 'Rust + egui': For performance enthusiasts
- Unity
	- Make a Unity editor to make this file
		- [[All Published Notes/Unity/Tools/Markdown File Generator - HowToAddPackage\|Markdown File Generator - HowToAddPackage]]
- 




---

[^1]: 
[^2]: 

