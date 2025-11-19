---
title: "Install Pelican Panel on Alpine Linux"
date: 2025-11-19
draft: true
tags: ["linux","alpine linux","game servers", "pelican", "nginx", "Proxmox LXC", "Proxmox", "Container"]
categories: ["Linux Guides"]
author: "Tanner Anderson"
showToc: false
TocOpen: false
hidemeta: false
comments: false
description: "Tested on Alpine 3.22.2"
canonicalURL: "https://tannerte.ch/blog/pelicanalpine"
disableShare: true
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---

## TODO: Formatting

Install PHP, Nginx, MariaDB, and dependencies:

`apk update`

```apk add php83 php83-gd php83-mbstring php83-bcmath php83-xml php83-curl php83-zip php83-intl php83-fpm php83-fileinfo php83-dom php83-tokenizer php83-session php83-sodium php83-pdo php83-simplexml php83-pdo_sqlite php83-xmlreader nginx curl tar unzip composer mariadb mariadb-client```

Follow official guide from https://pelican.dev/docs/panel/getting-started/#create-directories--downloading-files

Dont run the composer install, just run the command at the bottom of the page:
`COMPOSER_ALLOW_SUPERUSER=1 composer install --no-dev --optimize-autoloader`

Remove the default NGINX configuration:
`rm /etc/nginx/http.d/default.conf`

Grab the nginx config from https://pelican.dev/docs/panel/webserver-config/ ensuring you adjust the highlighted lines and place it at:
`/etc/nginx/http.d/pelican.conf`

Comment or remove the following lines from `/etc/nginx/http.d/pelican.conf`:
```
1 server_tokens off;

[...]

27 ssl_session_cache shared:SSL:10m;

[...]

47 fastcgi_pass unix:/run/php-fpm83/php8.3-fpm.sock;
```

Adjust /etc/php83/php-fpm.d/www.conf to replace:
```
28 user = nginx
29 group = nginx

41 listen = /run/php-fpm83/php8.3-fpm.sock

53 listen.owner = nginx
```
Proceed to https://pelican.dev/docs/panel/panel-setup/ ensure you replace any instance of www-data with nginx

### Database Setup

Run /etc/init.d/mariadb setup

Comment line 10 of /etc/my.cnf.d/mariadb-server.cnf (skip-networking)

Enable (at boot) and start the services we installed:

```
rc-update add nginx
rc-update add php-fpm83
rc-update add mariadb
rc-service php-fpm83 start
rc-service nginx start
rc-service mariadb start
```
There should be no errors at this stage.
Follow the official guide: https://pelican.dev/docs/panel/advanced/mysql/

Run the web installer at: https://yourdomain/installer
| Driver | Selection |
|--|--|
| Database | MariaDB |
| Cache | Filesystem |

### Queue driver
Paste the top command provided by the installer but do not paste the bottom one. As always ensuring you change any occurance of www-data to nginx.

Create the file /etc/init.d/pelican-queue copying the one at this link: https://tannerte.ch/scripts/pelican-queue

Make it executable, autostart, and run now:
```
chmod +x /etc/init.d/pelican-queue
rc-update add pelican-queue
rc-service pelican-queue start
```

For Session Driver pick between Filesystem and Database. Most likely you will want to go with Filesystem.

In future when you want to update the panel just follow the official guide: https://pelican.dev/docs/panel/update/ again replacing www-data with nginx when prompted.

## Finished!
Now proceed to adding your nodes. If you'd like to run your nodes on alpine as well check out my guide here: (TBA)