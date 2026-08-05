---
layout: post
title: Install Homer in a Container
categories: [homelab, lxcs]
tags: [proxmox, linux, container, homepage]
after-content: [disclaimer-notice.html]
---

[Homer](https://github.com/bastienwirtz/homer) is a simple, static homepage/dashboard for your homelab services. This is a quick walkthrough for deploying it in a Proxmox LXC.

## Create a Proxmox LXC

Create a new LXC using the `debian-12-standard_12.2-1_amd64.tar.zst` template. Homer is a static site, so it doesn't need much: 1 core and 512MB of RAM is plenty. Give it a static IP address on whichever VLAN your other homelab dashboards live on, and start the container once it's created.

## Install Homer

Connect to the LXC's console and log in as root, then install nginx and download the latest Homer release:

~~~
apt update
apt install unzip nginx
wget https://github.com/bastienwirtz/homer/releases/latest/download/homer.zip
unzip homer.zip -d /var/www/homer
cd /var/www/homer
cp assets/config.yml.dist assets/config.yml
systemctl start nginx
systemctl enable nginx
nano /etc/nginx/sites-available/homer
~~~

Add the following server block:

~~~
server {
    listen 80;
    server_name homer;
    root /var/www/homer;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
~~~

> **_PREFERRED METHOD:_** Instead of changing the port Homer is configured to listen on, you could also configure a reverse proxy using haproxy or nginx proxy manager.

Test the nginx config, enable the site, and reload:

~~~
nginx -t
~~~

~~~
ln -s /etc/nginx/sites-available/homer /etc/nginx/sites-enabled/
rm /etc/nginx/sites-enabled/default
systemctl reload nginx
~~~

Get the LXC's IP address to confirm you can reach it:

~~~
hostname -I
~~~

Browse to that IP address and you should see Homer's default dashboard. From there, edit `/var/www/homer/assets/config.yml` to add links to your own services. See Homer's own [documentation and config reference](https://github.com/bastienwirtz/homer) for all the available options.
