---
layout: post
title: Custom Linux Commands
date: 2021-07-06 18:00:00 +0200
# intro: SSH connect through PowerShell.
categories: [Linux, Unix, Terminal]
---

Whenever we need to access data on our on-premise server, then we need to connect with a VPN to the company network.

We use FortiClient, and they have a great Windows client to easily login get access to the data we need.

Because I am using Linux, I sometimes have to take longer routes to be able to use a service as simple as this.

Luckely because of open-source, another developer created a service where you can connect to a FortClient VPN service through the terminal. This package is called [openfortivpn](https://github.com/adrienverge/openfortivpn).

After installing the package, you can set up a config file where you do not need to pass all the necessary parameters. This already saves a lot of time. My next issue is that I always have to pass the file through as a parameter and I did not save this file in my home or root folder as this contains some sensitive info, thus it is better to save it in a deeper folder.

I always have to enter the following command when connecting to FortiClient:

```sh
$ sudo openfortivpn -c /etc/openfortivpn/my-config 
```

This already is too much for me, so I have created a custom command where I only enter the following into my terminal:

```
$ start_forticlient
```

It is actually very easy to create your own custom Linux shell commands.

## How To:

Open your terminal and make sure you are at the home folder. To verify that you are at your home folder, simply type `cd`.

Now we will create our custom commands script. We do not want anyone else to see the file as this will be in the background, so we will create a hidden file. A hidden file starts with a dot (`.`). Run the following command to create a new empty file:
```sh
$ touch .my_custom_commands.sh
```

Then open the file is a text editor and inser the following line to test it:
```sh
#!/bin/bash

# prints the input
function print_my_input() {
  echo 'Your input: ' $1
}
```

* The first line is a convention used while writing Shell scripts which gives information to the Shell for using the appropriate interpreter.
* `#` is used to write comments. This will help you remember your command if you write too many.

Save the file and close it. Then in your terminal enter the following:

```sh
$ print_my_input My name is Hanno
```

The `$1` in the script means that it can accept a parameter. This can be anything after the command that you enter. It will simply see it as a string and print out the string in the terminal.

If this works, then you can start creating your own commands!

**But!** There are still a few settings we need to adjust to make sure your commands will always work, even after you restart your system.

## Set File Permissions

Any file created in Linux only has read permissions. Sometime you want to run commands that need more elevated permissions, thus we will set "executable" permissions for the file to prevent any issues in the future. Run the following command:

```sh
$ chmod +x .my_custom_commands.sh
```

## Make commands available in terminal

If you close the terminal and open a new, then enter our custom command. You will receive an error message that the command does not exist.

This happens because we did not load the script in when the terminal starts, thus the terminal will never recognise the command.

Depending if you are using bash or zsh, open either `~/.bashrc` or `~/.zshrc` with `sudo nano`. If you do not know what zsh is then open `sudo nano ~/.bashrc`.

Scroll down to the bottom of the file and enter the following:
`source ~/.my_custom_commands.sh`

Save the file and the close it.

Close your terminal and open a new one and then run the `print_my_output` command, you will then see the `Your input:` output. This means that the terminal now loads the script!

## You are done!

You can now create even more custom commands that you wish to run. I have then immediately created my custome command to run OpenFortiVpn. Below is how the simple command looks.

```sh
# Start FortiClient...
function start_forticlient() {
    sudo openfortivpn -c /etc/openfortivpn/my-config 
}
```

*Please Note: Always be careful when creating custom commands as some of them can be malicious.*

This simple trick make my life a lot easier and I love it!