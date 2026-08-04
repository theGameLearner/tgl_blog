---
{"dg-publish":true,"permalink":"/all-published-notes/homelab/023-diff-checker-offline/"}
---

created: 2026-08-01
updated: 2026-08-04

### Idea
Whenever I am asking AI to edit a code, I use [diffchecker.com](https://www.diffchecker.com/). The problem is, I do not want to risk any scenario where something like password or auth credentials are pasted here by default.

I want a service which does the same task as it, but offline.
I also want the difference to be character wise, as many find differences line by line, but we may encounter the scenario where there are minor changes in characters in line and we miss some change with multiple character changes in a single line.

#### Options
The software(s) I found good enough to achieve this are:
- `Differ`: emergingtravel->differ
- `text-diff-view`: ghcr.io->kaishuu0123->text-diff-view

But these are again dependent on online updates, what if some setup changes? I thus decided to go for a static page over `monaco-editor` so I can run it locally, completely offline.
We can also run it with the scripts residing online, but I would like the luxury of being able to use the service without depending on internet.

> [!Warning]
> Using `monaco-editor` to directly use as a diff check gave unexpected and wrong results. Attempting to fix it failed, so I will shift my approach to `text-diff-view` docker file.

Steps for the options tried:
- [[All Published Notes/Homelab/023a diff checker offline - monaco-editor\|023a diff checker offline - monaco-editor]] (Failed)
- [[All Published Notes/Homelab/023b diff checker offline - text-diff-view\|023b diff checker offline - text-diff-view]] (Success)




---

[^1]: 
[^2]: 

