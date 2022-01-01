---
layout: single
title:  "Install Microsoft Teams on VDI."
date:   2019-09-19 15:08:31 +0000
categories: jekyll update

---

With Microsoft Teams set to replace Skype for Business in the future, more and more businesses are deciding to make the move and deploy Teams.

Typically Teams has been installed directly into the user profile which means it couldn’t be placed into the master image, but back in April Microsoft published this [article] [machinewide], opening the door for a machine wide installer, suitable for VDI.

Its worth taking note that Microsoft [doesn’t officially support] [nosupport] Audio and Video meetings on VDI. This support comes direct from vendors with VMware announcing official Teams support in Horizon on [version 7.9] [vmwaresupport] (be it with some hefty minimum vCPU recommendation of 4vCPU)

The install is pretty easy. you can grab the installer from either [here] [32bit] (32bit) or [here] [64bit] (64bit) and install it onto your gold master image using the following command.

```
msiexec /i <path_to_msi> /l*v <install_logfile_name> ALLUSER=1

e.g. msiexec /i c:\Teams_windows_x64.msi /l*v TeamsLog.log ALLUSER=1
```

Take note of the “ALLUSER=1” switch, this isn’t a spelling mistake, so be sure not to type the more commonly seen “ALLUSERS”.

The installer runs a pre check to make sure its being installed onto a VDI based machine. This pre check looks for registry keys relating to the major virtualization vendors. If it doesn’t find the keys it will produce an error message.

![image-title-here](/assets/images/teamsvdi/1.png)
{% include figure image_path="/assets/images/teamsvdi/1.png" alt="teams" caption="teams" %}

A Process Monitor capture reveals the regs keys its expecting:

![image-title-here](/assets/images/teamsvdi/2.png)

```
HKLM\SOFTWARE\Citrix\PortICA
HKLM\SOFTWARE\VMware, Inc.\VMware VDM\Agent
```


Simply adding the required registry key will allow the installer to complete, with or without the actual agent installed.

[machinewide]: https://docs.microsoft.com/en-us/microsoftteams/teams-for-vdi
[nosupport]:   https://docs.microsoft.com/en-us/microsoftteams/teams-for-vdi
[vmwaresupport]: https://docs.vmware.com/en/VMware-Horizon-7/7.9/rn/horizon-79-view-release-notes.html
[32bit]: https://teams.microsoft.com/downloads/desktopurl?env=production&plat=windows&download=true&managedInstaller=true
[64bit]: https://teams.microsoft.com/downloads/desktopurl?env=production&plat=windows&download=true&managedInstaller=true&arch=x64