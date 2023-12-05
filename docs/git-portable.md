---
tags: 
- portable-git
- vscode-portable 
---

# Using PortableGit with VSCode Portable

I changed devices and ended up having to do this again. Since I completely forgot how and failed to document this the last time, here are the field notes. 

File > Preferences > Profiles > Portable

From the error msg, click the cog > Manage extension

![Error Pop-up](assets/images/git-portable/git-portable_001.jpg)

Click the cog to go into extension settings.

![Git Extension Settings](assets/images/git-portable/git-portable_002.jpg)

Search for path to bring up relevant settings. Click the cog next to 'Git Path' to make sure the setting is being synced to the profile. 

![Search Path](assets/images/git-portable/git-portable_003.jpg)

Then click "Edit in settings.json" to open the settings page for this profile. 

![Sync Setting](assets/images/git-portable/git-portable_004.jpg)

Add the path to the git binary, save and reload the extension. 

![Settings.json](assets/images/git-portable/git-portable_005.jpg)

You should be all good now. 

!!! note
    You will not be able to use git in the integrated terminal but you should still be able to use the Git extension. 

Reference: 
[visual studio code - Use portable VSCode with portable Git - Stack Overflow](https://stackoverflow.com/questions/71515762/use-portable-vscode-with-portable-git)
