---
layout: single
title:  "How to deploy a PowerShell script as a Win32 Windows application via Microsoft Endpoint Manager/Intune"
date:   2022-06-20 10:00:00 +0000
categories: MEM PowerShell Win32 Intune
permalink: DeployPowershellAsAWin32App
tags:
  - MEM
  - PowerShell
  - Win32 App
toc: true
---
A quick blog on the syntax required to deploy a PowerShell script as a Win32 Windows application via Microsoft Endpoint Manager/Intune.

There are a number of ways to deploy a PowerShell script with MEM. You can deploy a script on its own, [the “traditional” way](https://docs.microsoft.com/en-us/mem/intune/apps/intune-management-extension), or you could look at [Proactive Remediation](https://docs.microsoft.com/en-us/mem/analytics/proactive-remediations), for a more reactive approach. The method detailed here is more suited to deploying a PSAppDeploy package or a custom script installer you’ve written yourself.

# Syntax

To deploy a PowerShell script via a Win32 application, edit the script name within the below command and use it as your ***install command*** when creating your application.

```powershell
Powershell.exe -NoProfile -ExecutionPolicy ByPass -File .\ScriptName.ps1
```

**Note:** This command will work with or without the `-NoProfile` parameter.

# Example

## Script to deploy

In this example, we will deploy the below script via MEM/Intune. The script creates the folder `C:\ProgramData\Example Script`, then creates a new text file called `Output.txt` with the content “Script executed” within.

```powershell
if (-not (Test-Path "$($env:ProgramData)\Example Script"))
{
    Mkdir "$($env:ProgramData)\Example Script"
}
Set-Content -Path "$($env:ProgramData)\Example Script\Output.txt" -Value "Script executed!"
```

## Prepare for upload

Save the script and package it into an .intunewin file using the [Microsoft Win32 Content Prep Tool](https://go.microsoft.com/fwlink/?linkid=2065730).

![createIntunewin.gif](assets/images/DeployPowerShellApp/createIntunewin.gif)

## Create and deploy the application

To create the application in MEM, go to *Apps*, *All Apps*, then select *Add*. Select *Windows (Win32)* from the App type drop down, then press *Select*.

![selectIntunewinFile.gif](assets/images/DeployPowerShellApp/selectIntunewinFile.gif)

Enter the relevant information on the *App Information* page, then select *Next*.

![appInfo.png](assets/images/DeployPowerShellApp/appInfo.png)

On the Program page, enter the install command `Powershell.exe -NoProfile -ExecutionPolicy ByPass -File .\script.ps1`. For the sake of this example, enter the same for the uninstall command. If you really wanted an uninstall script, you could use `Remove-Item 'C:\ProgramData\Example Script' -force`

![program.png](assets/images/DeployPowerShellApp/program.png)

Set the requirements relevant to your environment and select *Next*. On the detection rule page select *Manually configure detection rule* from the Rules format drop down menu. Select *Add* and enter the following details:

**Rule Type**: *File*.

**Path**: *C:\ProgramData\Example Script.*

**File or folder**: *Output.txt*.

**Detection method**: *File or folder exists*.

**Associated with a 32-bit app on 64-bit clients**: *No*.

Select *Ok,* then *Next.*

![detect.png](assets/images/DeployPowerShellApp/detect.png)

Next through the remaining pages and assign the app to a group. Select *Create* to finish packaging the app.

## Confirm deployment

The app will now deploy and once executed, the script will create the file  `Output.txt` in `C:\ProgramData\Example Script\`. Navigate to this folder to confirm the script has executed and the app has installed successfully.

![deployed.gif](assets/images/DeployPowerShellApp/deployed.gif)