---
{"dg-publish":true,"permalink":"/all-published-notes/unity/plugins/internet-access-check/"}
---

created: 2026-09-05
updated: 2026-09-05

link: https://github.com/tglGames-Plugins/Internet_Access_Check

A plugin to confirm if internet access is available. 
We often have issues where we don't know if the app is unable to reach the server or app is unable to connect online. This tool tells us the internet reachable status so we can easily debug the issue.

##### How to use
Add [InternetCheck.cs](https://github.com/tglGames-Plugins/Internet_Access_Check/blob/main/Runtime/Scripts/InternetCheck.cs) to any Game Object in scene and it will fire events and check for internet connection. The system works by trying to access multiple public URLs and fetching the head of the response, if we get head, it means we area connected to internet.









---

[^1]: 
[^2]: 

