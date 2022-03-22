---
title: How to Run a Rust Server in Docker on AWS
author:
  name: Hieu Huynh
  link: https://github.com/hhuynh15
date: 2022-03-21 12:10:00 -0700
categories: [Guides]
tags: [Instructional, Guides, Cloud, AWS, Linux, Docker, Rust]
image: 
  src: /rOSM02X.png
  width: 767
  height: 384
---

## Introduction

This guide will teach you how to run your own Rust server on the cloud. This guide assumes you already have some Linux knowledge and that you will be able to manage your server on your own after everything is set up. This is made for people who are not familiar with Docker or cloud services. Although we will be using AWS for this guide, the instructions here can be applied to any cloud service provider or even on your own on dedicated server.

## What's Docker and why Docker?

Docker is a system for running services such as apps in a containerized setting. This will give you the freedom to control the resources used by your servers and will also make it easier for you to expand to multiple servers in the future if you so wish. It will also allow you to freely move your server between devices and restore from backups. If you wish to learn more about Docker and how it works you can read more from [here](https://www.docker.com/resources/what-container/).

## Pre-setup

The only thing you will need is an AWS account. If you don't have one yet you can make one [here](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&client_id=signup&code_challenge_method=SHA-256&code_challenge=QJtklNBj78wdUxQgKDGEbr12Unt45QcxmWE-rLR_Rcs).

## Launching a Cloud Instance

- Once logged into your AWS account you can use the search bar at the top of the page and search for "EC2". 
- Select the first option and it will take you to the EC2 page. 
- Click the large orange button that says "Launch instances" on the top right. 
- Scroll down until you see "Ubuntu Server 18.04 LTS (HVM), SSD Volume Type" and select it. 
- You can hit next on the next two pages until you reach the "Add Storage" page.
- Here you change the size of your instance to 30 GiB.
- Now you can hit "Review and Launch" and then launch your new instance.
- A screen will pop up asking you if you want to "Select an existing key pair or create a new key pair".
- Since this is our first time launching an instance, select "Create a new key pair".
- Set whatever name you want for the key pair name.
- Press "Download Key Pair" and it will go to your downloads folder.
- You might have to wait several minutes before your instance is launched and is accessible.

## Installing Docker and Launching Your Rust Server

- In order to access your new instance you will need to use the key pair you created and downloaded earlier.
- You will need to modify the permissions of the private key so that you can use it.
- If you are on Linux, simply navigate to the folder where your key file is located and type the following command:
```bash
sudo chmod 400 {private key name}.pem
```
- On Windows you will have to modify the permissions of the file by right clicking the file then selecting properties and under the security tab hit "Advanced".
- On the new window that pops up you must remove permissions from "Users". This is how it looks on my desktop:

![Advanced Window](/hv0ncC1.png/posts/20190808/mockup.png){: width="586" height="114" }
