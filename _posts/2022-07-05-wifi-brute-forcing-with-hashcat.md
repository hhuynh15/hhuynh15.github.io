---
title: How to Perform a Brute Force Attack on a Wifi Network
author:
  name: Hieu Huynh
  link: https://github.com/hhuynh15
date: 2022-07-05 04:21:00 -0700
categories: [Guides]
tags: [Instructional, Guides, Networking, Hacking, Wifi, Offensive, Hashcat]
image: 
  src: /QmHbTU6.pngrOSM02X.png
  width: 1200
  height: 675
---

## Introduction
---

This guide will teach you some basic penetration testing skills for wireless networks. I hope to also demonstrate how having a weak password can leave the most secure networks vulnerable.

## Pre-setup
---

In order to follow along with this guide you will need a few things to get started. First you will need a wifi adapter that is capable of using monitor mode. If you don't know, you will have find out by searching online for your wifi adapter's chipset and seeing if it is able to utilize monitor mode. The next thing you will need will be an installation of Kali Linux. Although Kali is not the only operating system of performing these tasks, many of the tools we will be using today are already present in Kali so it makes the process smoother. If you choose to use Kali make sure your wifi adapter is compatible with Kali Linux before proceeding. You can do a quick google search to find out if your wifi adapter is compatible with Kali. And the last thing you will need will be a powerful enough GPU to perform the brute forcing required to crack these hashes.

## Preparing our tools
---

In your Kali installation make sure your wifi adapter is being detected by your operating system by typing in `iwconfig`. It should show up as so:

![Advanced Window](/6K9kKWF.png){: width="559" height="177" }

If something similar does not show up when typing `iwconfig` you might need to do some troubleshooting like making your adapter drivers are installed properly. Many times a quick reboot will solve most issues. If it works then we are ready to move on to the next step.

Type in the following commands:

```bash
sudo apt update
sudo apt upgrade
sudo apt install hcxdumptool
sudo apt install hcxpcapngtool
```

I've separated each command rather than combine them all into one so that we can go over what each command does. `sudo apt update` will update the repositories on the OS so when you make a call to install a library it has the latest references when grabbing a download link. `sudo apt upgrade` will download and install any updates to all the repositories and libraries installed on your OS. `sudo apt install hcxdumptool` will install the first tool we will be using to capture packets. And finally `sudo apt install hcxpcapngtool` will install the tool we will use to convert the packets we capture to a file that hashcat will be able to read and decrypt.