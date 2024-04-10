# 3P2 Tutorial for Hello-Server

# Install and Setting up a UFW Firewall
## What is UFW (Uncomplicated Firewall)

UFW is a program that makes it a little easier to manage a [netfilter](https://netfilter.org/) firewall. UFW actually manages nftables or iptables, which make use of netfilter which is part of the Linux kernel.
## Install UFW
```bash
#  before we start update your pages.
sudo pacman -Syu
# next install ufw Firewall
sudo pacman -S ufw
```
## Enable UFW Service
We can enable `ufw.services` using `systemctl`
```bash
# enable ufw.service with --now to indicate running at the same time.
sudo systemctl enable --now ufw.service
```

>[!warning]
>It is ok to enable *UFW Service*, BUT **DON'T ENABLE firewall** for now!!!
>Because you haven't set rules, you may lose SSH connection to your Digital Ocean Droplet!

## configuration steps
We need to make sure our server is safe so we can deny all incoming traffic and then add other rules to allow legitimate traffic to get in.
The outbound traffic is probably harmless, we just allow all outgoing traffic
```bash
# deny all incoming traffic, we need to then add some rules allow some traffic to come in.
sudo ufw default deny incoming
# allow all outgoing traffic
sudo ufw default allow outgoing
```
Enable some rules to let necessary traffic in, like ssh from port 22, http from port 80, and https from port 443
```bash
# allow all ssh traffic in
sudo ufw allow ssh
# but limit ssh malicious access like Brute Force Attack 6 attempts in 30s.
ufw limit ssh
# allow http and https traffic
sudo ufw allow http
sudo ufw allow https
```
Check your rules and status
```bash
sudo ufw status verbose 
```
You shall see 
```
sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22                         LIMIT IN    Anywhere                  
80                         ALLOW IN    Anywhere                  
443                        ALLOW IN    Anywhere                  
22 (v6)                    LIMIT IN    Anywhere (v6)             
80 (v6)                    ALLOW IN    Anywhere (v6)             
443 (v6)                   ALLOW IN    Anywhere (v6)             


```


# Create new service backend
Download `hello-server` binary file to you computer
## Upload backend binary file `hello-server` to your digital ocean droplet using SFTP
```bash
# connect through sftp 
sftp your_droplet
# using put to send the file to droplet
put hello-server
```
put it into `/usr/sbin` or `/usr/bin` or`/bin` directory
```bash
# move it into `/usr/sbin` or `/usr/bin` or`/bin`
sudo mv /home/your_username/hello-server /usr/bin/hello-server
# change ownership if necessary
sudo chown root:root /bin/hello-server
# make it executable
sudo chmod ug+x /bin/hello-server
```
## Write a service file
create and edit service file
```
sudo vim /etc/systemd/system/backend.service
```
add content
```bash
[Unit]
Description=Hello Server
After=network.target

[Service]
Type=oneshot
ExecStart=/home/your_username/bin/hello-server 

[Install]
WantedBy=multi-user.target
```
## Start your service
```bash
sudo systemctl daemon-reload
sudo systemctl start backend
```
# Adjust Nginx
Since we are going to open a new port to run the backend, so we have to change nginx settings
remember last time we have our configuration file in `/etc/nginx/sites-enabled/another.conf` (This is what we put our nginx configuration file last time. Yours may be different.)
```bash
# edit configruation file
sudo vim /etc/nginx/sites-enabled/another.conf
```

In the file we need to add two  locations for 1. `/echo`, 2. `/hey`
In the settings, we need to use reverse proxy to direct `/echo`,`/hey` traffic to `http://127.0.0.1:8080`. 

And then 'proxy_set_header' directives are used to pass headers from the original request, to the proxy server.
```bash
server {
    listen 80;
    listen [::]:80;
    root /web/html/nginx-2420;
    location / {
        index index.php index.html index.htm;
    }
    # here are the new added configuration.
    location /hey {
    # Define the reverse proxy settings
        proxy_pass http://127.0.0.1:8080/hey;
    }
    location /echo {
    # Define the reverse proxy settings
        proxy_pass http://127.0.0.1:8080/echo;
    }
    
}

```



# Test backend
```
curl http://24.144.93.244/hey
```
[image](./hey.png)
```
curl -X POST -H "Content-Type: application/json" \
  -d '{"message": "Hello from your server"}' \
  http://24.144.93.244/echo
```
You shall see this. If you use a postman do test.
[image](./echo.png)
# Good Job!