ACIT 2420 - Assignment 3 Part 1
# Instructions to serve an HTML document from a fresh Arch Linux server using nginx
This tutorial is right for you if you have just created a fresh Arch Linux server running on a DigitalOcean droplet to serve a simple HTML document.

## Step 1: install vim
In order to write files, scripts, or code in our Arch Linux server, we are going to use `vim` which is a text editor. 
**Install `vim` by typing `sudo pacman -S vim`.**
`sudo` elevates the command to be run as the root user. `pacman` is the name of the package manager used in Arch Linux, which will manage the installation of software and its dependencies. The `-S` option denotes that we want to install the packages and its dependencies from the remote repositories that host them. 

## Step 2: install nginx
We are going to use `nginx` to host our web server in order to serve our HTML content.
**Install `nginx` by typing `sudo pacman -S nginx`.**
`sudo` elevates the command to be run as the root user. `pacman` is our package manager. `-S` means we want to install the packages and dependencies from remote repositories. 

## Step 3: start the nginx server to test if it is working
**Start the `nginx` server by typing `sudo systemctl start nginx`.**
`systemctl` is used to manage system services. In the command above, we are technically starting `nginx.service`, but we can omit the `.service` portion as it is the default. 

We can check if nginx is indeed working by using `systemctl`.
**Check if the `nginx` service is running by typing `systemctl status nginx`.**

## Step 4: create a new directory as our project root 

