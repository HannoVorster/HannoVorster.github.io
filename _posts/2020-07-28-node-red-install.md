---
layout: post
title: Node-Red Install
date: 2020-07-28 18:03:59 +0200
categories: [Node-RED, IoT, Raspberry Pi]
---

Node-Red is a great tool for wiring together hardware devices, APIs and online services.

This guide wil show you how to install Node-Red onto your Raspberry Pi.

## Required Equipment
*  Raspberry Pi 1, 2, 3, 4
* SD Card (8 GB recemmended depending on Rasbian version)
* 12V Power supply
* Ethernet cable

The next post will show you how to enable WiFi on a wireless Raspberry Pi.

## Install Node-Red
Make sure you are SSH into your Raspberry Pi.

I am working on a Windows computer, thus I use PowerShell to SSH into my raspberry Pi.

### Updating your Raspberry Pi
Assuming you have Rasbian installed.

Open a terminal and enter the following:

```zsh
$ sudo apt update && apt upgrade upgrade -y
```

This will update your Pi to the latest version.

### Download Node-Red
To download install Node-Red, enter the following command:

```zsh
$ bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```
This will install the latest version of Node-Red along with other libraries and packages needed to run Node-Red.

Note: Everytime you reboot or turn your Pi off and then off, you need to manually start Node-Red with the node-red-start command.

Run the following command to let Node-Red autostart on boot:

```zsh
$ sudo systemctl enable nodered.service
```

Reboot your Pi with `sudo reboot`.

You will lose your SSH connectin, wait for about 20 seconds and SSh back into your Pi.

### Access Node-Red
To access Node-Red, check your Pi host name with hostname -I.

Open your favorite browser and enter your hostname with port number 1880.

Example: `192.168.9.100:1880`   

## Optional: Admin Password
This is an optional step, but highly recommended as this will prevent someone else on the same network to access Node-Red.

Stop Node-Red with the following command:

```zsh
$ node-red-stop
```
This will allow you to make changes to the Node-Red configuration file.

Go into the Node-Red folder:

```zsh
$ cd ~/.node-red
```

Install node-red-admin:

```zsh
$ npm install node-red-admin -g
```

Create a new password by entering the following:

```zsh
$ node-red admin hash-pw
```
You will be prompted to enter a password.