---
{"dg-publish":true,"permalink":"/all-published-notes/hardware/mobile/termux/changing-termux-font/"}
---

created: 2025-08-02

### Introduction
If we want to use a `.ttf` font file for mobile's Termux, we need to store the file in a location that we can access easily.
I am using [Kode Mono](https://fonts.google.com/specimen/Kode+Mono) font from Google font as it is one of my favourite fonts on a terminal.
### Steps
As this article is about Termux, I hope you have it. Otherwise, kudos to you to learn something new and potentially not so useful article.
#### Getting the font on mobile
Download and extract the font file (`.ttf`) you want to use as a font in the terminal, it is easier if you use a monospace font as terminal needs to show information which is not easy to read with fonts like [Style Script](https://fonts.google.com/specimen/Style+Script). I am keeping my file in `documents/fonts/Kode_Mono/` and the font I will be using is in `documents/fonts/Kode_Mono/static/KodeMono-bold.ttf`.

#### Termux Font
Termux stores the ttf file it uses in `.termux/` folder under `.termux/font.ttf` file. This is a set name and we need to replace this file to use a different font.
In my Termux, I will run the code for moving my bold font to Termux's font file (`mv storage/documents/fonts/Kode_Mono/static/KodeMono-bold.ttf ~/.termux/font.ttf` ) which will replace the old font file with the new font file.

#### Applying the changes
To apply this change, we need to reload the Termux application: `termux-reload-settings`
In case your reload does not work for any reason, you can kill the app and re-start it to see the new fonts.

![Changing Termux Font.png|300](/img/user/All%20Published%20Notes/Hardware/Mobile/Termux/Changing%20Termux%20Font.png)


# Footnotes
[^1]:
[^2]:

