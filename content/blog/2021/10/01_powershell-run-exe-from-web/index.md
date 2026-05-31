---
title: 'PowerShell: Run Exe from Web'
description: "Ever needed to run an executable across many devices (maybe even spanning organizations)? Check this out if you're newer to powershell."
summary: "Ever needed to run an executable across many devices (maybe even spanning organizations)? Check this out if you're newer to powershell."
category: Scripting
tags: [Powershell]
date: 2021-10-01
slug: powershell-run-exe-from-web
authors:
    - "wizardtux"
---

So there are times that you may need to run an executable (exe) across many devices, and maybe those devices are across several organizations or not on a Windows Domain. With this script you can upload the file to a central web server and the script will download the file and run it on devices.

Note: This script was created for our previous Remote Monitoring and Management system (RMM) though could be modified to be ran in an organization on remote machines using PSRemoting.

```powershell
param(
    [Parameter(Mandatory=$true)]
    [string]$packageUri, # url to download the exe from
    [Parameter(Mandatory=$true)]
    [string]$packageName, # what to name the exe
    [string]$packageSwitches #any package switches
) #end param

$path = "C:\"
$folder = "Tools"
$fullPath = -join($path,$folder,"\",$packageName)

function Test-Folder {
    if (-Not (Test-Path -Path $folder)) {
        New-Item -Path $path -Name $folder -ItemType "directory" -Force
    }
} #end Test-Folder

function Get-Installer {
    if (Test-Path -Path $fullPath) {
        Remove-Item -Path $fullPath -Force
    }
    Invoke-WebRequest -Uri $packageUri -OutFile $fullPath
} #end Get-Installer

Test-Folder #verify the folder exists
Get-Installer #download a new installer
#run the install
Start-Process $fullPath -ArgumentList $packageSwitches -NoNewWindow -Wait

exit 1
```
It's a pretty simple script. Not flashy but it works.

If you want more or need help, stop over at our forums.