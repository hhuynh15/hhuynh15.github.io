---
title: Let’s Build a Home Server Rack
author:
  name: Hieu Huynh
  link: https://github.com/hhuynh15
date: 2022-11-05 04:21:00 -0700
categories: [Guides, Homelabs]
tags: [Instructional, Guides, Networking, Linux, VMWare, Cisco, Homelab, Server]
image: 
  src: /GpzzpOJ.jpg
  width: 1200
  height: 675
---

# Introduction
---

Building a home server rack is a great way to expand your knowledge in network and server administration, and gain hands-on experience with enterprise-level technologies. By using Linux, VMware, and Cisco switches you can build a robust and flexible home lab environment that simulates a real-world IT infrastructure. In this guide, we will outline the steps involved in building a home server rack and provide detailed information on the hardware and software components used.

# Gathering the Hardware
---

The first step in building a home server rack is to gather the necessary hardware. You will need:

- A server rack: This can be a standalone rack or a cabinet that can accommodate multiple servers and switches. The size of the rack will depend on the number of servers and switches you plan to use. For a home lab, a 10U server rack should be sufficient.
- Servers: You can use one or multiple servers, depending on your needs. For a home lab, an Intel NUC or a small form-factor PC can work well. The servers should have sufficient RAM and storage to accommodate the virtual machines you plan to run.
- Switches: You can use any switch but today we will be focusing on Cisco switches as they are the most commonly used switches in the industry and are a good place to start practicing routing and switching protocols. For a home lab, you can use Cisco Catalyst 2960 and if you have Juniper switches on hand you can use those as well. For the most part these switches are the same but may have different command line syntax. These switches are compact and have a limited number of ports, making them ideal for a home lab environment.
- Cables: You will need Ethernet cables to connect the switches and servers to each other, and power cables to power the servers. The cables should be of the appropriate length to reach from the servers to the switches, and from the switches to the power outlets.

When building your rack and racking your servers and switches. I have a few pointers for best practices:


- Build your rack from the ground up: Stack your servers starting from the ground and place subsequent servers on top. Have your switches at the top of the rack. Larger servers will require two people to help rack and should be racked first. Don’t be afraid to ask for help.
- Know the physical dimensions of your components: Keep in mind the length and depth of your server rack. Although most racks will allow you to adjust its dimensions. You don’t want to be rebuilding an entire rack to make a two inch adjustment.
- Leave room to cable manage: It’s easy to want to maximize the space in your rack and not leave a lot of room in the back of your rack. But having good cable management will make it easier for you to make modifications to your rack in the future. It also allows for better ventilation for your equipment.
- Have cable management accessories: Cable management accessories are great for easier cable management. I would recommend a 1U cable management unit and plenty of velcro ties. I would not recommend using zip ties as they are difficult to work with when you need to make modifications.

Here is an example build from my own server rack:

![Advanced Window](/Fiysyq0.png){: width="700" height="400" }

# Installing Linux and VMware
---

Once you have the hardware, you can install Linux and VMware on the servers. For Linux, you can use Ubuntu or CentOS, both of which are popular distributions for server administration. You can download the ISO image of the distribution from the official website and install it on the servers using a bootable USB drive.

For VMware, you can use vSphere, which is a virtualization platform that allows you to run multiple virtual machines on a single physical server. You can download a trial version of vSphere from the VMware website and install it on the servers. After installation, you can configure vSphere to manage the virtual machines and allocate resources, such as RAM and storage, to each virtual machine.

# Configure the Switches

Let’s start by connecting to our switch either through a console connection or SSH/telnet. Next enter global configuration mode by typing:

```bash
Switch> enable
Switch#configure terminal
```

Now let’s configure the hostname and create a new vlan for our switch:


```bash
Switch(config)#hostname homelab-switch
homelab-switch(config)#vlan 10
```

You can give your vlan an easily recognized name like this:

```bash
homelab-switch(config-vlan)#name public
```

Now that we have our vlan created let’s start assigning a port to the vlan:

```bash
homelab-switch(config-vlan)#exit
homelab-switch(config)#interface FastEthernet 0/1
homelab-switch(config-if)#switchport mode access
homelab-switch(config-if)#switchport access vlan 10
```

If you want to assign multiple ports to a vlan at once. We would do it like so:

```bash
homelab-switch(config-if)#exit
homelab-switch(config)#interface range FastEthernet 0/2-10
homelab-switch(config-if)#switchport mode access
homelab-switch(config-if)#switchport access vlan 10
```
Next let’s configure an IP address to our vlan:

```bash
homelab-switch(config-vlan)#exit
homelab-switch(config)#interface vlan 10
homelab-switch(config-if)#ip address 10.10.0.2 255.255.255.0
```

Make sure your configurations are correct:

```bash
homelab-switch(config-if)#exit
homelab-switch#show ip interface brief
Interface   IP-Address	OK?	Method	Status	Protocol
Vlan10    10.10.0.2	YES	manual	up	down
```

Congratulations we have just configured our Cisco switch for our homelab. There is much more to configuring a switch for your network such as configuring link aggregation, spanning tree, and access control lists. But for now this is sufficient for our basic usage.

# Making the Hash File and Cracking It

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

If you have any additional information that might help narrow down the attack be sure to take advantage of it as it could make the difference between waiting a week for the password to get cracked to a few hours. To give an example, an offensive security expert went around sniffing for wireless passwords and was able to crack a good amount of them by only guessing ten digit numbers cause many people were using their phone numbers as their passwords. Below we will show examples of different attack vectors and ways you can narrow your search.

# Different Brute Force Attacks

## Using a Wordlist

A wordlist is a preset list of common passwords also known as a dictionary attack. A list of common wordlists can be found in Kali under `/usr/share/wordlists/` but it can also be downloaded from [here](https://github.com/00xBAD/kali-wordlists).

```bash
hashcat -m 22000 hash.hc22000 wordlist.txt
```

## Brute Forcing Alphanumeric and Special Character Passwords

The following command will brute force a password with digits, lowercase, upercase and special characters:

```bash
hashcat -m 22000 hash.hc22000 -1 ?d?l?u?s -a 3 ?1?1?1?1?1?1?1?1
```

## Rule-based Attack

This is similar to a dictionary attack but the commands look a bit different:

```bash
hashcat -m 22000 hash.hc22000 -r rules/best64.rule cracked.txt.gz
```

This will mutate the wordlist with best 64 rules, which comes with the hashcat distribution. Change as necessary and remember, the time it will take the attack to finish will increase proportionally with the amount of rules.

# Getting Your Results and Conclusion

If your attack was a success you will find the cracked password in a `.potfile` or you can type out the same cracking parameters again but tagging `--show` at the end.

These are some of the most basic techniques a hacker can use to try to get into your network. Even if WPA/WPA2 is considered a secure way of protecting your network. It can easily be broken into if you don't have an equally secure password as well. Over time cracking and brute forcing methods will only get faster and more efficient so the need for a strong password becomes increasingly necessary.

As demonstrated above it is easier to crack a sophisticated but short password than it is to crack a long but easy to remember password. A password containing 8 random characters like this `1S7Hd8@3` will be incredibly hard for a human to guess but incredibly easy for a computer to guess. Even a password that uses a common phrase like `DrinkLots_OfWater825` will take a computer hundreds of years to guess. So in this instance, size does matter.

