# Purpose of the Tutorial
Set up a fresh Arch Linux server running on DigitalOcean to serving the demo document included below.
# Nginx
[nginx](https://en.wikipedia.org/wiki/nginx "wikipedia:nginx") (pronounced "engine X"), is a free, open-source, high-performance HTTP [web server](https://wiki.archlinux.org/title/Web_server "Web server") and reverse proxy, as well as an IMAP/POP3 proxy server, written by Igor Sysoev in 2005. nginx is well known for its stability, rich feature set, simple configuration, and low resource consumption.

# Steps of setting
## Install Nginx
In Arch Linux, we can use this command to install nginx
```bash
# before we do installation, it is high recommended to refresh and upload the packages
sudo pacman -Syu
# install ngnix
sudo pacman -S nginx
```
## Install vim editor
We have to install vim editor to change configuration file and html
```bash
# install vim
sudo pacman -S vim
```
## Test Nginx 
We can run a test of Nginx, in the bash, using this command
```bash
nginx
```
we can set see the page for now
![[Pasted image 20240403085058.png]]
It is running on the default port 80

## Enable nginx service using systemctl
We have to enable nginx service using systemctl, so it can run automatically every time when we start our droplet
```bash
sudo systemctl enable nginx
# we can check whether it is enabled, using 
sudo systemctl status nginx
# if it shows enabled, then it's cool
```
## Create our directory for project root
We can use this path `/web/html/nginx-2420`, or you can create your own as you wish
Here `mkdir` is the command to make directory.
`-p` option tells it to create parent directory if necessary.
We need `sudo` because we are making directory on root directory
```bash
sudo mkdir -p /web/html/nginx-2420
```

## Create our own directory with the content below
Under the directory of `/web/html/nginx-2420`, we can make our own html file.
You can name your html file as you wish, here I just use `index.html`
```bash
sudo vim /web/html/nginx-2420/index.html
```
Paste in the content in the following block, or you can paste in/create your own html content.
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        * {
            color: #db4b4b;
            background: #16161e;
        }
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
            font-family: sans-serif;
        }
    </style>
</head>
<body>
    <h1>All your base are belong to us</h1>
</body>
</html>
```
Press `Esc` and type in `:wq` to save the file.
## Create a separate server block in another configuration file
1. Create the `sites-enabled` and `sites-available `directory
```bash
sudo mkdir -p /etc/nginx/sites-enabled
sudo mkdir -p /etc/nginx/sites-available
```
2. We create another configuration file. You can name your own configuration file
```bash
sudo vim /etc/nginx/sites-enabled/another.conf
```
3. Fill in server block, be careful if you set a different name for your html file, you have to change location part, filling your own name of html.
```bash
server {
    listen 80;
    listen [::]:80;
    server_name 24.144.93.244
    root /web/html/nginx-2420;
    location / {
        index index.php index.html index.htm;
    }
}
```
use `Esc` + `:wq` to save
## Setting up Nginx configuration file
The default setting file is in `/etc/nginx/nginx.conf`
```bash
sudo vim /etc/nginx/nginx.conf
```
1. Delete default setting this part, or like me, just comment them out
```bash
#server {
#listen       80;
#server_name  localhost;
 #charset koi8-r;
#access_log  logs/host.access.log  main;
#location / {
#	 root   /usr/share/nginx/html;
#	index  index.html index.htm;
#}
```
2. Include the new configuration file, in http part past in `include sites-enabled/*;` like this
```bash
http {
    ...
    include sites-enabled/*;
}
```
use `Esc` + `:wq` to save

3. To enable a site, simply create a symlink:
	If you have different configuration file, just change the configuration file below
```bash
sudo ln -s /etc/nginx/sites-available/another.conf /etc/nginx/sites-enabled/another.conf
```

## Restart Ngnix.service
After all is set, we need to restart it 
```bash
sudo systemctl restart nginx
```

Good now go to your server address, and it is done.

# Debugging
If there are error message showing, we can use `nginx -t` to show the configuration problems.
Like there is an error message about 
```bash
2024/04/03 17:06:47 [warn] 1450726#1450726: could not build optimal types_hash, you should increase either types_hash_max_size: 1024 or types_hash_bucket_size: 64;
```
You can change the configuration file `types_hash_max_size: 1024` into `types_hash_max_size: 4096` (it is under `http` block) It should be solved.