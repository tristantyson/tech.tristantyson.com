---
layout: single
title:  "Install and Configure OneDrive on VMware Horizon for Non-Persistent Pools"
date:   2022-05-18 10:00:00 +0000
categories: VMware Horizon OneDrive
permalink: VMwareHorizonOneDrive
tags:
  - VMware Horizon
  - OneDrive
toc: true
---
I recently migrated our on-premise VMware Horizon environment over to VMware Horizon Cloud on Azure. As part of the migration we stopped using DEM folder redirection and instead implemented OneDrive and Known Folder Moves. The move made a lot more sense as it mimicked our physical device configuration and was more cloud focused in the new cloud based environment.

In this post I will go over the installation and configuration needed to get this up and running.

For reference, my environment uses AppVolumes for application management, as well as DEM and FSLogix for profile management.
{: .notice--info}

# Supported Scenarios

- Virtual desktops that persist between sessions.
- Non-persistent virtual desktops that use Azure Virtual Desktop.
- Non-persistent virtual desktops that have FSLogix Apps or FSLogix Office Container

# Install OneDrive

You can download the latest version of the stand alone OneDrive sync client from the [OneDrive release notes](https://support.microsoft.com/en-gb/office/onedrive-release-notes-845dcf18-f921-435e-bf28-4e24b95e5fc0) website. 

Do not install OneDrive as part of the general Office 365 installation. Make sure its excluded in your O365 configuration XML file.
{: .notice--danger}

Typically OneDrive installs into the user profile during the first user logon. This obviously causes issues in non-persistent environments where a user logs onto a different virtual machine each time. Microsoft gives the ability to install the sync client on a per-machine basis, meaning all users share a single local installation. Installing the sync client this way on your master image greatly improves the logon process.

To install the OneDrive sync client in per-machine mode; open an elevated command prompt, change directory to the folder where you downloaded the latest client, run the OneDriveSetup.exe with the /allusers switch.

```
OneDriveSetup.exe /allusers
```

![install.gif](/assets/images/VMwareHorizon-OneDrive/install.gif)

# Disable Automatic Updates

OneDrive automatically checks for updates via a scheduled task. To stop OneDrive from automatically updating, we need to delete this scheduled task.

On your master image, after you have installed/manually updated OneDrive, open Task Scheduler.

Under ‘Task Scheduler Library' there is an entry called 'OneDrive Per-Machine Standalone Update Task'.

Right click this task, and select delete.

![disableautoupdate.gif](/assets/images/VMwareHorizon-OneDrive/deleteautoupdate.gif)

# Fix the Grey Cross (X) Issue

When using OneDrive you might notice some strange issues where files will not sync and, more visibly, a grey cross (X) will appear on all your OneDrive file icons.

![Untitled](/assets/images/VMwareHorizon-OneDrive/greycrossissue.png){: .align-center}

This issue is caused by the AppVolumes agent on the virtual machine.

To resolve, we need to add an exception to the AppVolumes sysvol.cfg file, which tells the AppVolumes agent to not interfere with the OneDrive sync client.

On your master image navigate to (and create if necessary) `%SVAgent%\Config\Custom`.

Create or modify the snapvol.cfg file so that it contains:

`exclude_registry=\REGISTRY\MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\SyncRootManager`

![Untitled](/assets/images/VMwareHorizon-OneDrive/snapvolcfg.png)

# Suggested policies

In order to provide a seamless user experience, there are a number of policies you should set that control how OneDrive behaves. 

## Silently sign in users to OneDrive

Enabling this setting will cause the OneDrive sync client to automatically sign in without prompting the user to enter their account credentials. The client will use the windows login credentials to sign in.


The virtual machine needs to be Azure AD joined for this policy to take effect.
{: .notice--warning}

**Set via Group Policy:**

Location: *Computer Configuration\Policies\Administration Templates\OneDrive\.*

Policy Name: *Silently sign in users to the OneDrive sync client with their Windows credentials.*

Setting: *Enabled.*

![Untitled](/assets/images/VMwareHorizon-OneDrive/SSOPolicy.png)

**Set via Registry:**

```
[HKLM\SOFTWARE\Policies\Microsoft\OneDrive]"SilentAccountConfig"="dword:00000001”
```

## Files on Demand

Enabling this setting tells the OneDrive sync client to use the Files on Demand Feature. Files on Demand helps to save storage space by only downloading files when they are needed. A placeholder file is presented to the user but is only actually downloaded to the machine once opened.

You can see the status of your files based on the status icon.   

![Untitled](/assets/images/VMwareHorizon-OneDrive/iconstatus.png){: .align-center}

- The blue cloud means that it is an *online only* file, not actually downloaded onto the machine yet.
- A green tick means that it is an *locally available file.* The file has been downloaded locally onto the machine.
- A green tick with a solid background means that it is an *always available* file. This file will always been downloaded locally onto the machine.

**Set via Group Policy:**

Location: *Computer Configuration\Policies\Administration Templates\OneDrive\.*

Policy Name: *Use OneDrive Files On-Demand.*

Setting: *Enabled.*

![Untitled](/assets/images/VMwareHorizon-OneDrive/filesondemandpolicy.png)

**Set via Registry:**

```
[HKLM\SOFTWARE\Policies\Microsoft\OneDrive]"FilesOnDemandEnabled"="dword:00000001"
```

## Known Folder Move

Enabling this setting automatically redirects the users Documents, Pictures, and Desktop folders to OneDrive.

**Set via Group Policy:**

Location: *Computer Configuration\Policies\Administration Templates\OneDrive\.*

Policy Name: *Silently move Windows known folders to OneDrive.*

Setting: *Enabled.*

Options:
- Tenant ID: *Enter your O365 tenant ID here*
- Show notifications: *No*

![KFMPolicy.png](/assets/images/VMwareHorizon-OneDrive/KFMPolicy.png)

**Set via Registry:**

```
[HKLM\SOFTWARE\Policies\Microsoft\OneDrive]"KFMSilentOptIn"="1111-2222-3333-4444"
[HKLM\SOFTWARE\Policies\Microsoft\OneDrive]"KFMSilentOptInWithNotification"="dword:00000000"
```
Note: 1111-2222-3333-4444 represents your tenant ID.

# User Experience

Installing and configuring OneDrive in this way allows for a seamless user experience. A user can logon to a virtual machine and OneDrive will automatically sign-in, synchronise, and redirect folders, all without any user interaction or prompts.

![fulllogonprocess.gif](/assets/images/VMwareHorizon-OneDrive/fulllogonprocess.gif)