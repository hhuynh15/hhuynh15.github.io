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
- When selecting which CPU type it's important to select one that can accomodate the size of your server.
- Since this is for demonstration purposes I will be selecting the "r5a large" type which can handle about 16 players.
- After seclecting your CPU and memory hit next on the bottom right until you reach the "Add Storage" page.
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
sudo chmod 400 {Private key name}.pem
```

- On Windows you will have to modify the permissions of the file by right clicking the file then selecting properties and under the security tab hit "Advanced".
- On the new window that pops up you must remove permissions from "Users". This is how it looks on my desktop:

![Advanced Window](/hv0ncC1.png){: width="586" height="114" }

- Select "Users" and hit "Remove".
- Go back to your AWS instances page and look for your "Public IPv4 DNS". It will look something like this:

![IPv4 Public DNS](/2O9Zc8O.png){: width="1715" height="70" }

- You will need the "Public IPv4 DNS" to connect to your instance.
- Once completed you can open your terminal or command prompt and navigate to the folder with your private key.
- Enter the following command to connect to your instance:

```bash
ssh -i {Private key name}.pem ubuntu@{Public IPv4 DNS}
```

- If everything is successful you should be greeted with a "Welcome to Ubuntu 18.04.6 LTS" message in your terminal.
- First thing we will be doing is running the two following commands:

```bash
sudo apt update
sudo apt upgrade
```

- This will update and upgrade our repositories and repository libraries.
- Next we will install Docker:

```bash
sudo apt install docker.io
```

- Let's launch our Rust server for the first time!

```bash
sudo docker run --name rust-server didstopia/rust-server
```

- What this does is, first download the lastest rust-server image from Docker Hub (a public repository of Docker images), then it starts a new rust-server container, giving it the name "rust-server".
- This might take a while as you are starting your server for the first time.
- To stop your server you can use:

```bash
sudo docker stop rust-server
```

- To see a full list of Docker commands  you can use:

```bash
sudo docker help
```

## Configuring your Rust Server

- First create your server configuration file:

```bash
touch /rust.env
```

- Inside the file you will need to add your environment variables. Put the following and customize where necessary:

```
RUST_SERVER_STARTUP_ARGUMENTS=-batchmode -load +server.secure 1
RUST_SERVER_IDENTITY=rustserver
RUST_SERVER_SEED=12345
RUST_SERVER_NAME=My Containerized Rust Server on AWS!
RUST_SERVER_DESCRIPTION=I made this!
RUST_RCON_PASSWORD=CHANGEME
```

- There are a lot more variables you can use to customize your servers. A full list can be found [here](https://hub.docker.com/r/didstopia/rust-server/).
- Before firing up your Rust server with your new environment variables file you must delete your old container:

```bash
sudo docker rm rust-server
```

- Next, run the following command:

```bash
sudo docker run --name rust-server -d -p 28015:28015 -p 28015:28015/udp -p 28016:28016 -p 8080:8080 -v /rust:/steamcmd/rust --env-file /rust.env didstopia/rust-server
```

We have added the `-d` flag, which will detach or daemonize the process, so that we wont get regular console outputs from the server anymore.

The `-p` flag will bind our local ports to the container ports, which will enable us to connect to the server from outside our network.

The `-v` flag is where the magic happens, as the `-v` flag translates to volume. Meaning it will mount a volume on the local filesystem inside the container. The first part is the host path (/rust in this case), and the second part is the container path (/steamcmd/rust, which is always the same).

Since Docker containers aren't persistent (they can't be permanently modified, changes aren't saved and so on), using a volume will now store both the Rust installation data, as well as the server data (level, users, blueprints, logfiles). Storing the Rust installation data on the host system means that when the container is restarted, it won't have to download the entire server again, but it can still update it if necessary.

The ``--env file`` flag simply loads our variables from the file we set up earlier.

To access the server console, we can do so with this command:

```bash
sudo docker logs -f rust-server
```

To stop the server, we would run:

```bash
sudo docker stop rust-server
```

To update to the lastest image by pulling from Docker Hub:

```bash
sudo docker pull didstopia/rust-server
```

If we get an error saying that a server with the name *rust-server* already exists, we can remove the container by running:

```bash
sudo docker rm -f rust-server
```