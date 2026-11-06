---
title: "Install Pelican Panel on Alpine Linux"
date: 2026-11-5
draft: false
tags: ["linux", "alpine-linux", "game-servers", "pelican", "nginx", "proxmox-lxc", "proxmox", "containers"]
categories: ["Linux Guides"]
author: "Tanner Anderson"
showToc: false
TocOpen: false
hidemeta: false
comments: false
description: "Tested on Alpine 3.24.4"
canonicalURL: "https://tannerte.ch/blog/pelicanalpine"
disableShare: true
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---
## 1. Install PHP, Nginx, MariaDB, and Dependencies

Update packages:

```sh
apk update
```

Install required packages:

```sh
apk add php85 php85-gd php85-mbstring php85-bcmath php85-xml php85-curl php85-zip \
  php85-intl php85-fpm php85-fileinfo php85-dom php85-tokenizer php85-session \
  php85-sodium php85-pdo php85-simplexml php85-pdo_sqlite php85-xmlreader \
  nginx curl tar unzip composer mariadb mariadb-client
```

Follow the official guide:
https://pelican.dev/docs/panel/getting-started/#create-directories--downloading-files

**Do not run** the default composer install command. Instead run:

```sh
COMPOSER_ALLOW_SUPERUSER=1 composer install --no-dev --optimize-autoloader
```

---

## 2. Configure Nginx

Remove the default Nginx configuration:

```sh
rm /etc/nginx/http.d/default.conf
```

Download and adjust the official Pelican Nginx config:
https://pelican.dev/docs/panel/webserver-config/

Save it to:

```
/etc/nginx/http.d/pelican.conf
```

Comment or remove the following lines in `pelican.conf`:

```nginx
1 | server_tokens off;
...
27| ssl_session_cache shared:SSL:10m;
```

Replace the unix: socket on line 47 with:
```nginx
47| fastcgi_pass unix:/run/php-fpm85/php8.5-fpm.sock;
```

---

## 3. Configure PHP-FPM

Edit `/etc/php85/php-fpm.d/www.conf` and adjust:

```ini
28| user = nginx
29| group = nginx
...
41| listen = /run/php-fpm85/php8.5-fpm.sock
...
53| listen.owner = nginx
```

Follow the panel setup guide:
https://pelican.dev/docs/panel/panel-setup/

Replace **all** instances of `www-data` with `nginx`.

---

## 4. Database Setup

Initialize MariaDB:

```sh
/etc/init.d/mariadb setup
```

Comment out line 10 in:
```
/etc/my.cnf.d/mariadb-server.cnf
```
Specifically:
```
skip-networking
```

Enable and start services:

```sh
rc-update add nginx
rc-update add php-fpm85
rc-update add mariadb

rc-service php-fpm85 start
rc-service nginx start
rc-service mariadb start
```

There should be **no errors** at this point.

Follow the MySQL guide:
https://pelican.dev/docs/panel/advanced/mysql/

---

## 5. Web Installer

Open the installer:
```
https://yourdomain/installer
```

| Driver   | Selection   |
|----------|-------------|
| Database | MariaDB     |
| Cache    | Filesystem  |

### Queue Driver
Copy the **top** command provided by the installer (not the bottom one). Again, replace any `www-data` occurrences with `nginx`.

Create the queue service file:
```
/etc/init.d/pelican-queue
```
Using the version from:
https://tannerte.ch/scripts/pelican-queue

Make executable and enable on boot:

```bash
chmod +x /etc/init.d/pelican-queue
rc-update add pelican-queue
rc-service pelican-queue start
```

### Session Driver
Choose between **Filesystem** and **Database**.

I suggest **Filesystem**.

---

## 6. Updating the Panel

For upgrades, follow:
https://pelican.dev/docs/panel/update/

Replace any reference to `www-data` with `nginx`.

---

## Finished!
You can now proceed to add your nodes. If there are any issues please submit an issue [here](https://github.com/tannertechnology/blog)

