# Adding a Firewall using UFW and a Reverse Proxy Server
This tutorial is right for you if you just finished the previous tutorial on how to host your own web server using nginx.

In this tutorial, we will place a backend binary file onto our system. Then, we will create a new service file to run the backend. We will also add reverse proxy to backend, and also configure a firewall using UFW. 

## Step 1: place the given binary file from our local server to the remote server
Ensure you have downloaded the `hello-server` binary file onto your local computer/server. Note the file path relative to your home directory. For example, I am using macOS and my home directory is `/Users/kenneth`.

We will use the binary file to handle HTTP requests to our web server.

**Open a Terminal or equivalent (such as Windows Powershell). Then, type the following command making sure you replace `<these-fields>` with your own unique details.**
**`sftp -i .ssh/<droplet-key-name> <username>@<remote-ip-address>`**
If successful, you should see an `sftp` prompt. 

`sftp` stands for Secure File Transfer Protocol. Using `sftp`, we will be able to securely copy a file (like the hello-server binary file) from our local device to the remote droplet.

## Step 2: use `put` to copy the file from your local host to the remote server
We are going to use the `put` command to copy the file from the local device to the remote server.
We are going to copy it to `/usr/local/bin`.
**Using the `sftp` prompt, type the following command:**
**`put <location-of-binary-file-on-local> /usr/local/bin`**

For example, my command would look like:
`put /Users/kenneth/hello-server /usr/local/bin`

`/usr/local/bin` is an appropriate directory to store hello-server as it stores user-configured binary files. 

**Exit `sftp`.**

**SSH back into your Arch droplet: `ssh -i .ssh/<droplet-key-name> <username>@<remote-ip-address>`.**

**Use `cd` to navigate to the `/usr/local/bin` folder, where we saved the `hello-server.service` file: `cd /usr/local/bin`.**

The `hello-server` binary file needs to be executable in order to be used. We can give it execute permissions using `chmod`. 
**Use `chmod` to give `hello-server` executable privileges: `sudo chmod u+x hello-server`.**

You can check the file's permissions by using `ls -l` in the file's directory. 

## Step 3: write a new service file
**Navigate using `cd` to the `/etc/systemd/system` directory, which is an appropriate place to put our user-created service file.**

**Create a new file by typing: `sudo touch hello-server.service`.**
You can choose the name for your service file. For example, you could name it `hello-backend.service` or `backend.service`.

**Use `vim` to open the file in a text editor: `sudo vim hello-server.service`. To enter `INSERT` mode, press `i`.**

Inside the file, type the following script:
```
[Unit]
Description=service to run the hello-server backend

[Service]
ExecStart=/usr/local/bin/hello-server

[Install]
WantedBy=multi-user.target
```

**Write to the file by first exiting insert mode by pressing `ESC`. Then, enter command mode by typing `:`. Type `w` to write to the file, then type `q` to quit the text editor.**

Unit files can have three sections. Since we are writing a service file, one of the sections is called [Service]. Therefore, our three sections are: [Unit], [Service], and [Install].

The [Unit] section provides metadata for the service. For example, we are adding a description of the service, which could be helpful in identifying it during debugging. 

The [Service] section includes an `ExecStart` directive, which indicates that this service will start the file: `/usr/local/bin/hello-server`. 

The [Install] section includes a `WantedBy` directive with a value of `multi-user.target`. This indicates that we want this service to be available to multiple users. 

## Step 4: start the hello-server service
**Start the `hello-server.service` by entering the following into the shell: `sudo systemctl enable --now hello-server.service`.**

Since we are making system changes, we need to temporarily elevate our privileges using `sudo`. 

The `systemctl` utility allows us to manage services. The `enable` command will make the service start on startup. The `--now` flag in conjunction with `enable` will start the service now. 

**Check that your service started properly by typing: `systemctl status hello-server.service`.** We don't need `sudo` for this command since we are not making any changes.

If the service started properly, you should see `enabled` on the `Loaded` line, and `active (running)` on the `Active` line. 

## Step 5: edit the nginx-2420.conf file
**Use `cd` to navigate to `/etc/nginx/sites-available`, which we created in the previous tutorial.**

**After navigating to the `sites-available` folder, open the `nginx-2420.conf` file in vim (our text editor): `sudo vim nginx-2420.conf`.**
You can check your current directory using `pwd` (present working directory).

**Type `i` to enter INSERT mode in vim.**

**Make changes to the server block to implement a reverse proxy.**
Your `nginx-2420.conf` will look like this: 
```
server {
    listen 80;

    location / {
        index index.html;
        root /web/html/nginx-2420;   
    }

    location /hey {
        proxy_pass http://127.0.0.1:8080;
    }

    location /echo {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

**Exit INSERT mode by pressing `ESC`. Then, write to the file and quit vim by typing: `:wq`.** 

The server block contains instructions for `nginx`. 
`listen 80` means that the server will be listening for requests made to port 80, which is the default port for HTTP traffic. 

`location /` indicates what to do when the request is trying to access the `/` route. We want to pass HTTP requests to `/hey` and `/echo` to our proxied server, so we need to specify a `proxy_pass` directive inside a location block.

The hello-server backend has been configured to run on 127.0.0.1:8080 (port 8080), so this is the value we are specifying for the `proxy_pass` directive.

## Step 6: check if the nginx configurations are valid
We can check to see if the changes to the configuration is valid. 

**Check if the config is valid using: `sudo nginx -t`.**
We need `sudo` because part of the test involves opening `/run/nginx.pid`, which requires elevated permissions. 

If the configuration files are valid, you should see a message saying "test is successful". 

## Step 7: reload the nginx service
**Reload the `nginx` service so the changes to the configuration file is reflected. You can do this by typing: `sudo systemctl reload nginx`.**

The reload command asks the specified unit(s) to reload their configuration. 

If you see an error message, try debugging the issue specified in the error message. For example, there may be a syntax issue where a brace is missing. 

## Step 8: reload the systemd configuration
Systemd is a service manager which runs as the first process. Since we added our own `hello-server.service`, it is a good idea to reload the systemd configuration.

**Use `daemon-reload` to reload systemd configuration files: `sudo systemctl daemon-reload`.**

## Step 9: install `ufw` 
**Install `ufw` using: `sudo pacman -S ufw`.**
Since we are installing a new package, we need elevated privileges using `sudo`. 

`pacman` is the name of the package manager in Arch, which will install the package and its dependencies. 

`-S` effectively means we want to install the specified package onto the system. 

## Step 10: configure firewall using `ufw`
We can use `ufw` to configure a firewall. 

Since we are using ssh to access the remote droplet from our local device, we need to allow ssh connections through the firewall. 
**Allow ssh connections through the firewall by typing: `sudo ufw allow 22`.**

`ufw` also allows management using names. For example, we could have typed `sudo ufw allow ssh` instead. We could have also used `sudo ufw limit ssh` to limit excessive unsuccessful attempts to access our server using ssh. 

**Allow http connections through the firewall by typing: `sudo ufw allow http`.** 
Alternatively, we could have typed `sudo ufw allow 80`. 

You can check which connections have been configured to be allowed by typing `sudo ufw status verbose`. In order to use `ufw`, we need to run the command as root, which necessitates `sudo`.  

## Step 10: enable ufw
**Enable the ufw firewall using `sudo ufw enable`.**

Typing `sudo ufw status verbose` should show `Status: active`. 

## Step 11: test the connection 
Using your host machine, open a Terminal or equivalent. 

**Test the following commands:**
`curl http://<your-remote-ip-address>`
`curl http://<your-remote-ip-address>/hey`
```
curl -X POST -H "Content-Type: application/json" \
  -d '{"message": "Hello from your server"}' \
  http://<your-remote-ip-address>/echo
```

Consult the screenshots in the git repo for what the output should look like. 