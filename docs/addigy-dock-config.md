---
tags: 
- dock-config
- dockutil
- addigy
- outset
---
# macOS Dock Configuration with Dockutil, Outset and Addigy

Once upon a time, I configured docks with dockutil and outset and deployed them via Addigy. 

Addidy does a lot of the heavy lifting as it relates to automating the onboarding experience, but it does have some limitations. At the time, you *could* configure the dock in the Addigy interface, but the config was fixed - it could not be changed or customized by the user after the fact (but it could be redeployed). 

Since that was not ideal, I originally set this up to round out the sharp edges in our provisioning workflow. There are multiple utilities that can do this, but with a little bit of bash, I found this to be the simplest. **Dockutil** customizes the dock, **Outset** automates the script, **Addigy** packages them together into Custom Software (now called Smart Software) and automates the deployment.

Once the utility dependencies are installed, you can run a condition script in Addigy to check for and install any applications in your script that may be missing - to prevent a wonky dock. 

###### Dependencies 

Dockutil
Python
Outset 

###### Scripts

The specific scripts I deployed are in the [shell-scripts](https://github.com/damitasalmon/shell-scripts) repo, under bash/dockutil. They leverage bash, but previous versions of dockutil leveraged python instead of swift, so (naturally) the older they are, the less likely they'll work provided the current dependencies. It's mostly the same from the best of my recollection, but double-check the dockutil syntax just in case. 

##### Using with Addigy, specifically

The one caveat with this, is that if you apply it to an already existing policy, all users will be affected - even old ones that already have their dock just as they like it. Although it's only set to execute at first login post deployment - again, not ideal - so I only deployed this on a *separate* policy with brand new or "refreshed" devices. As users left the organization and the devices were returned to us to be redeployed to new users - the config would slowly proliferate the inventory. Overtime, less monitoring is needed (see notes). 

###### Installation Command

Addigy can prepopulate the installation with the "Install Command" button - but the bottom half is the important bit. 

```
#Add these with Install Command button for the correct dir

/usr/sbin/installer -pkg "/Library/Addigy/ansible/packages/Default Dock (4.0.0)/dockutil-3.0.2.pkg" -target /


/usr/sbin/installer -pkg "/Library/Addigy/ansible/packages/Default Dock (4.0.0)/outset-3.0.3.pkg" -target /
  
  

#Change permission of the script
chmod +x ./setDock-defaultDock.sh

#Move the files to their appropriate directory
cp setDock-defaultDock.sh /usr/local/outset/login-once

```

###### Condition for install

You can easily build this in Addigy and customize to the apps you want to deploy. 


###### Removal Command

Do not use this unless you want it removed. 

```
/bin/rm -Rf '/usr/local/outset/'
/bin/rm -Rf '/usr/local/bin/dockutil'
```

###### Notes

Okay, one more caveat. Occasionally, this would not work - the script would not deploy. This has more to do with the quirks of software deployment in Addigy than anything else (see Addigy Notes). Addigy as a product, continuously improves, so we experienced fewer bugs over time. In the event that it did not deploy automatically - on schedule, or we needed it to deploy immediately, I just manually pushed the Custom Software item like you would any other custom install on a device. 

##### Helpful Resources
[Apple Dock Documentation](https://developer.apple.com/documentation/devicemanagement/dock)<br />
[GitHub - kcrawford/dockutil: command line tool for managing dock items](https://github.com/kcrawford/dockutil)<br />
[GitHub - chilcote/outset: Automatically process packages, profiles, and scripts during boot, login, or on demand.](https://github.com/chilcote/outset)<br />
[How to use dockutil in a script | AppleShare IT](https://appleshare.it/posts/use-dockutil-in-a-script/)<br />
[Automating a Great On-boarding Experience - YouTube](https://www.youtube.com/watch?v=I-J5ty7SQls)<br />
