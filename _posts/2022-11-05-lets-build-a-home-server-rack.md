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

For VMware, you can use vSphere, which is a virtualization platform that allows you to run multiple virtual machines on a single physical server. This is great if you have very few machines to work with and the same principles of provisioning and configuring physical servers apply just the same to virtual machines. You can download a trial version of vSphere from the VMware website and install it on the servers. After installation, you can configure vSphere to manage the virtual machines and allocate resources, such as RAM and storage, to each virtual machine.

# Configure the Switch

Let’s start by connecting to our switch either through a console connection or SSH/telnet. Next enter global configuration mode by typing:

```shell
Switch> enable
Switch#configure terminal
```

Now let’s configure the hostname and create a new vlan for our switch:


```shell
Switch(config)#hostname homelab-switch
homelab-switch(config)#vlan 10
```

You can give your vlan an easily recognized name like this:

```shell
homelab-switch(config-vlan)#name public
```

Now that we have our vlan created let’s start assigning a port to the vlan:

```shell
homelab-switch(config-vlan)#exit
homelab-switch(config)#interface FastEthernet 0/1
homelab-switch(config-if)#switchport mode access
homelab-switch(config-if)#switchport access vlan 10
```

If you want to assign multiple ports to a vlan at once. We would do it like so:

```shell
homelab-switch(config-if)#exit
homelab-switch(config)#interface range FastEthernet 0/2-10
homelab-switch(config-if)#switchport mode access
homelab-switch(config-if)#switchport access vlan 10
```
Next let’s configure an IP address to our vlan:

```shell
homelab-switch(config-vlan)#exit
homelab-switch(config)#interface vlan 10
homelab-switch(config-if)#ip address 10.10.0.2 255.255.255.0
```

Finally, let’s enable inter-vlan routing and also make sure your configurations are correct:

```shell
homelab-switch(config-if)#exit
homelab-switch#ip routing
homelab-switch#show ip interface brief
Interface 	IP-Address	OK?	Method	Status	Protocol
Vlan10		10.10.0.2	YES	manual	up	down
```

Congratulations we have just configured our Cisco switch for our homelab. There is much more to configuring a switch for your network such as configuring link aggregation, spanning tree, and access control lists. But for now this is sufficient for our basic usage.

# Setting up virtual machines and networking

With the servers and switches configured, it's time to set up the virtual machines and networking. This requires a good understanding of virtualization and networking technologies.

## Virtual Machines

To create virtual machines in VMware, you can use the vSphere client. It's a graphical user interface that allows you to manage virtualized environments. Here are the steps to create a virtual machine:

1. Open the vSphere client and log in to the vCenter Server.
2. Click on the "Create a new virtual machine" option in the Home screen.
3. Select the type of virtual machine you want to create: Custom, Typical, or Instant Clone.
4. Select the guest operating system you want to install on the virtual machine.
5. Allocate resources such as CPU, memory, and storage to the virtual machine.
6. Configure the network adapter for the virtual machine and connect it to a virtual switch.
7. Customize the virtual machine with additional hardware, such as a CD/DVD drive or additional network adapters.
8. Review the configuration options and complete the creation process.
(go more in depth about how to set up a vm with vsphere)

## Virtual Networking

The virtual networking in VMware is implemented through virtual switches. You can create virtual switches and connect them to physical switches to create virtual networks. The same principles of networking apply within virtual networks so what we did above with our real life Cisco switch we can configure the same type of network in vSphere. Here's how you can create a virtual switch in VMware:

1. Open the vSphere client and log in to the vCenter Server.
2. Navigate to the host you want to create the virtual switch on.
3. Click on the "Configure" tab and select "Networking."
4. Click on the "Add Networking" button and select "Virtual Switch."
5. Enter a name for the virtual switch and select the physical switch you want to connect it to.
6. Configure the settings for the virtual switch, such as VLANs, security policies, and traffic shaping policies.
7. Click on the "Finish" button to create the virtual switch.

## Troubleshooting and Testing 

After setting up the virtual machines and networking, it's important to test and troubleshoot the environment. This helps to identify and resolve any issues that might arise during deployment. Here are some of the common testing and troubleshooting tasks:

- Ping test: Use the `ping` command to test connectivity between the virtual machines and the physical network.
- Traceroute: Use the `traceroute` command to determine the path taken by packets from the source to the destination.
- VLAN Configuration: Verify that the VLANs are configured correctly on the switches by using the `show vlan` command on Cisco switches and `show interfaces [interface_name] switchport` command on Juniper switches.
- Firewall rules: Verify that the firewall rules are configured correctly on the virtual machines and switches.
- Router Configuration: Verify that the routing protocols, such as OSPF and BGP, are configured correctly on the routers and switches. You can use the `show ip route` command on Cisco routers to verify the routing information.
- Virtual Machine Configuration: Verify that the virtual machines are configured correctly and that they can communicate with each other and the physical network.
- Switch Configuration: Verify that the physical switches are configured correctly and that they are able to communicate with the virtual switches and the routers.

## Monitoring and Maintenance

Finally, it's important to monitor and maintain the home server rack environment to ensure that it operates optimally. Here are some of the common monitoring and maintenance tasks:

- Resource Monitoring: Monitor the resources of the virtual machines, such as CPU usage, memory usage, and disk space, to ensure that they are not running out of resources.
- Backup and Recovery: Regularly back up the virtual machines and switches to ensure that you can recover from any failures or data loss.
- Software Updates: Regularly update the software on the virtual machines, switches, and routers to ensure that they are running the latest security patches and bug fixes.
- Virtual Machine Management: Manage the virtual machines, including creating, modifying, and deleting virtual machines, as needed.

# Conclusion

Creating a home server rack using Linux, VMware, Cisco, and Juniper technology is a great way to build a powerful and flexible computing environment. However, it requires a good understanding of virtualization, networking, and system administration technologies. By following the steps outlined in this post, you can create a home server rack that is robust, secure, and scalable.

