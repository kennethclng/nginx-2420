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
We need to create a directory to contain the HTML content we want to host.
**Navigate to the root folder by typing `cd /`.**
`/` is the root folder. 

In this specific location, make the directories.
**Type `sudo mkdir -p /web/html/nginx-2420`.**
`mkdir` is the command used to create new directories. 
The `-p` option creates parent directories as long as no errors are detected. This will allow us to do this all in one command instead of creating the web directory, then the html directory inside of it, and then the nginx-2420 directory inside of that.

You can name the final directory something other than `nginx-2420`.

## Step 5: add the HTML file to the html/nginx-2420 folder
**Navigate to the folder we just created by typing `cd /web/html/nginx-2420`.** 
**Create a new file to hold the HTML content by typing `sudo touch index.html`. Then, open it in vim by typing `sudo vim index.html`.**
**To add text using vim, enter insert mode by typing `i`.**
Copy and paste the HTML code into the file.

**To save the file, first, exit insert mode by pressing `ESC`. Then, enter command mode by typing `:`. Finally, type `wq` to `-w` write text to the file and then `-q` to quit the vim editor.**

## Step 6: add a separate server block
**Navigate to the `/etc/nginx` folder by typing `cd /etc/nginx`.**

In this specific location, create two new directories.
**Type `sudo mkdir sites-available` and then `sudo mkdir sites-enabled`.**

**Navigate to the sites-available directory by typing `cd /etc/nginx/sites-available'.**
Here, we will create a config file containing our server block.

**Create a new file in this specific location by typing `sudo touch nginx-2420.conf`.**
**Open the file in vim by typing `sudo vim nginx-2420.conf`.**

Enter insert mode by typing `i` in vim.
**Type the following code into the text editor:**
```
server {
    listen 80;
    root /web/html/nginx-2420;
    location / {
        index index.html;
    }
}
```
You can change the port number to something other than 80, but port 80 is the default port nginx will be listening for.

**Write to the file and quit vim by exiting insert mode by typing `ESC`, then entering command mode by typing `:`, then typing `wq`.**

## Step 7: edit the /etc/nginx/nginx.conf file
**Navigate to the /etc/nginx folder by typing `cd /etc/nginx`.**
**Open the nginx.conf file. In the /etc/nginx directory, type `sudo vim nginx.conf`.**

At the end of the `http` block, insert code to include `sites/enabled`.
**Enter insert mode, then at the bottom of the http block, type: `include sites-enabled/*;`.**

Your http block should look like this:
```
http {
    ... some code here ...
    include sites-enabled/*;
}
```
In the next step, we'll make some more changes to the `nginx.conf` file.

## Step 8: comment out the existing default behaviour in the nginx.conf file
Still editing the nginx.conf file, comment out the existing server block to disable the default behaviour.
**Comment out lines of code by adding a `#` to the beginning of the line.**

When you're ready, write to the file and quit vim. 
Check that your nginx.conf file for syntax errors by typing `sudo nginx -t`. 
The `-t` option tests the configuration file without running it.

## Step 8: enable a site by creating a symlink
In order to enable a site, we have to create a symlink.
**In the command line interface, type `ln -s /etc/nginx/sites-available/nginx-2420.conf /etc/nginx/sites-enabled/nginx-2420.conf`.`**

The `ln` command makes links between files.

## Step 9: restart the nginx.service to reload the changes
**Type `sudo systemctl restart nginx`.**

## Step 10: access the webpage
Find the droplet's IP address by typing `ip address`.

**In a web browser, open the webpage by typing your ip address.**

If everything is working as intended, you should see your HTML document being served. 
