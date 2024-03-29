---
title: How to Set Up an AD Domain (Video Demonstration)
author:
  name: Hieu Huynh
  link: https://github.com/hhuynh15
date: 2023-03-21 08:24:00 -0700
categories: [Guides, Homelab, Active Directory]
tags: [Instructional, Guides, Networking, Active Directory, DHCP, DNS, RAS, NAT, Domain, Powershell, Windows 10, Windows Server]
---

{% include embed/youtube.html id='Z-vYzYUzMVQ' %}

# Description

For this guide I thought it would be best to demonstrate how to set up Active Directory with a video as trying to do so with a blogpost would make it very hard to follow along. Making it in a video form also allows me to more effectively explain many of the concepts that goes into creating a domain controller by explaining it visually via drawing tools. I hope you enjoy the video.

# Timestamps

[Installing Windows 2019 Server on VirtualBox 1:17 - 10:00](https://youtu.be/Z-vYzYUzMVQ?t=77)

[Configuring Network Adapters 10:10 - 12:54](https://youtu.be/Z-vYzYUzMVQ?t=610)

[Configuring the Domain Controller 12:58 - 24:22](https://youtu.be/Z-vYzYUzMVQ?t=778)

[Configuring Network Address Translation 25:15 - 28:48](https://youtu.be/Z-vYzYUzMVQ?t=1515)

[Configuring DHCP Server 31:00 - 35:13](https://youtu.be/Z-vYzYUzMVQ?t=1860)

[Creating Users w/ Powershell 35:24 - 51:29](https://youtu.be/Z-vYzYUzMVQ?t=2124)

[Creating a Windows 10 Client on VirtualBox 52:38 - 59:20](https://youtu.be/Z-vYzYUzMVQ?t=3112)

# Links
[Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)

[Server 2019 ISO](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)

[Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10ISO)

[Random Name Generator](https://www.randomlists.com/random-names?qty=1000)

# Script Used in the Video

```shell
$PASSWORD = "Password1"
$USER_NAMES = Get-Content .\names.txt

$ENCRYPTED_PASSWORD = ConvertTo-SecureString $PASSWORD -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_NAMES) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()
    Write-Host "Creating User: $($username)" -BackgroundColor Black -ForegroundColor Cyan

    New-AdUser -AccountPassword $ENCRYPTED_PASSWORD `
               -GivenName $first `
               -Surname $last `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_USERS,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
}
```
