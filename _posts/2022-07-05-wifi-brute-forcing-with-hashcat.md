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

In order to follow along with this guide you will need a few things to get started. First you will need a wifi adapter that is capable of using managed/monitor mode. If you don't know if your adapter is capable of that, you will have find out by searching online for your wifi adapter's chipset and seeing if it is able to utilize monitor mode. The next thing you will need will be an installation of Kali Linux. Although Kali is not the only operating system of performing these tasks, many of the tools we will be using today are already present in Kali so it makes the process smoother. If you choose to use Kali make sure your wifi adapter is compatible with Kali Linux before proceeding. You can do a quick google search to find out if your wifi adapter is compatible with Kali. And the last thing you will need will be a powerful enough GPU to perform the brute forcing required to crack these hashes.

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

## Scanning for networks

Let's start by disabling our network services so we can use our network adapter to scan for networks.

```bash
sudo systemctl stop NetworkManager.service
sudo systemctl stop wpa_supplicant.service
```

Next we will use the `hcxdumptool` we downloaded earlier to scan for networks.

```bash
sudo hcxdumptool -i wlan0 -o dumpfile.pcapng --active_beacon --enable_status=15
```

The first flag in this command `-i` will specify which interface we will be using to perform the scan. The `-o` flag will output the contents of the scan into a file. In this instance it will be the `dumpfile.pcapng` file. The `--active_beacon` flag will transmit a beacon to every collected ESSID from the collected ESSID list every few minutes. And finally the ``--enable_status=15` flag will display the following network information into the terminal: `EAPOL` `ASSOCIATION` `REASSOCIATION` `AUTHENTICATION` `BEACON`. You can always read more about the specific command using `man hcxdumptool`.

Let the tool run for a few minutes and capture information. After a while you can use the shortcut `Ctrl+C` to stop the scan and now we can start cracking.

## Making the Hash File and Cracking It

Next, we will need to convert the newly created pcap file we just made from the scan into a format that can be processed by hashcat.

```bash
sudo hcxpcapngtool -o hash.hc22000 -E essidlist dumpfile.pcapng
```

Agains the `-o` flag will output the file as `hash.hc22000` and the `-E` flag will output the collected ESSIDs into a list. Now that we have the necessary files we can start cracking. Remember to do this on a computer with a sufficiently powerful GPU otherwise it will take you hours to crack the simplest of passwords. Since we are attempting to penetrate our own network for this example, we have set the password to all digits to show the speed and efficiency at which we can crack a simple password like this.

Before we can start cracking we need to specify which network we want to crack by removing every other network we found in the last scan. Use the following command to help find the mac address:

```bash
sudo hcxdumptool --do_rcscan -i wlan0
```
Use `Ctrl-C` to exit the scan. On the left side you will see the BSSID of the network. That is what you will need to identify your network. Match it with the ESSID and write down the information before proceeding.

Open up your `hash.hc22000` file with a text editor, it should look something like this:

```
WPA*02*b38d2a7c87aa0864201c41eaac3bd074*20becd364387*3420037d4435*4b6569746857696669*082b00483ffa60261b82837a38916fb92e4e0d81608343c95f2d97a978c449ed*0103007502010a00000000000000000001eb021bf76d5b9a03965527c2796825b9e268e7cdccc4f4603dce98b894613612000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001630140100000fac040100000fac040100000fac020000*02
WPA*02*fdf01f941daddca2bd780d2f2a6f5711*20becd364387*3c846ae3aeb6*4b6569746857696669*72cc768ae878785bf30c22e54cc23e21362e7fbd3597d92e7bf43b88889b8ef7*0103007502010a0000000000000000fa12f4345f77049ab5a66ce61f600af4cdcf0c28104bcbbb9b17e941949249746dd2000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001630140100000fac040100000fac040100000fac020000*10
WPA*02*0b4838355821f9625ca3971d6e40c934*20becd364387*3c846ae3cfcd*4b6569746857696669*72cc768ae878785bf30c22e54cc23e21362e7fbd3597d92e7bf43b88889b8ef7*0103007502010a0000000000000000fa12f852203ce4dd9ff5196d583503e10c476f203bcf5d9223ba5cecfa434910f432000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001630140100000fac040100000fac040100000fac020000*10
WPA*02*9c465e97913d7c93ea37023392303428*20becd364387*6038e0edd011*4b6569746857696669*72cc768ae878785bf30c22e54cc23e21362e7fbd3597d92e7bf43b88889b8ef7*0103007502010a0000000000000000fa1225873e630b8cf46ea6eaac659603576b6dbd1e3e3640e2931ca37ce3f0b1d24d000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001630140100000fac040100000fac040100000fac020000*10
```

Delete all the lines that **do not** contain your target BSSID. Save the file and it will be what you will use to perform the attack.

We will demonstrate multiple attack vectors for cracking this hash. Firstly, we will try cracking only digits:

```bash
sudo hashcat -m 22000 hash.hc22000 -a 3 ?d?d?d?d?d?d?d?d
```

The first flag in this command, `-m` will specify the hashmode to use for this attack. Since we will be cracking WPA/WPA2 we will be specifying `22000` for the mode. You can find a full list of different hashmodes [here](https://hashcat.net/wiki/doku.php?id=example_hashes). The next flag is the `-a` flag and will specify the attack mode for this command execution. For this example we will be doing a brute force attack which will iterate through every possible variation to find the right combination so we will set the attack mode to `3`. You can read more about the different attack modes [here](https://hashcat.net/wiki/doku.php?id=hashcat). And finally the `?d` at the end of the command specifies which character type to try when performing the attack. There are four different character types: digits, uppercase, lowercase, and special characters. The `?d` denotes digits. We will try an alphanumeric attack in a later example.

## Different Brute Force Attacks

# Using a wordlist

A wordlist is a preset list of common passwords also known as a dictionary attack. A list of common wordlists can be found in Kali under `/usr/share/wordlists/` but it can also be downloaded from [here](https://github.com/00xBAD/kali-wordlists).

```bash
hashcat -m 22000 hash.hc22000 wordlist.txt
```

# Brute Forcing Alphanumeric and Special Character Passwords

The following command will brute force a password with digits, lowercase, upercase and special characters:

```bash
hashcat -m 22000 hash.hc22000 -1 ?d?l?u?s -a 3 ?1?1?1?1?1?1?1?1
```

# Rule-based Attack

This is similar to a dictionary attack but the commands look a bit different:

```bash
hashcat -m 22000 hash.hc22000 -r rules/best64.rule cracked.txt.gz
```

This will mutate the wordlist with best 64 rules, which comes with the hashcat distribution. Change as necessary and remember, the time it will take the attack to finish will increase proportionally with the amount of rules.