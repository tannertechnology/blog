---
title: "Run Splashtop for RMM on Linux"
date: 2023-04-012T05:55:08-07:00
draft: false
tags: ["linux","wine","RMM"]
categories: ["Wine Guides"]
author: "Tanner Anderson"
showToc: true
TocOpen: false
hidemeta: false
comments: false
description: "Tested on: Wine 6.14 staging | Wine-6.0.1 stable | Wine Staging 7.0 | Wine Staging 8.5"
canonicalURL: "https://tannerte.ch/blog/splashtop"
disableShare: true
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---

## Install Splashtop

Grab your Splashtop installer and click it to run it. Nothing will appear but it will install.  

Run `winetricks vcrun2015` to install a required redistributable.


## Test Splashtop

Head into the wine prefix you set (normally `~/.wine`) and find the directory `drive_c/Program Files (x86)/Splashtop/Splashtop Remote/Client for RMM/`


Double click on `clientoobe.exe` or run `wine clientoobe.exe`


In the window that opens click Options, open the Advanced tab, and change the renderer to Software. 


Head into your RMM tool and get the link you would normally click to launch Splashtop. Copy this link and navigate in a terminal to the directory containing clientoobe.exe. Run  


`wine clientoobe.exe -a "linkyoucopied" `


You should connect to the machine.  


## Create XDG association 
### The following instructions have only been tested on Ubuntu & Fedora

Create `~/.local/share/applications/st-rmm.desktop` with this content, replacing `username` on the Exec= line with your username:

```ini
[Desktop Entry] 
Encoding=UTF-8 
Exec=wine "/home/username/.wine/drive_c/Program Files (x86)/Splashtop/Splashtop Remote/Client for RMM/clientoobe.exe" -a %u 
Terminal=false 
StartupNotify=true 
Name=SplashtopRMM 
Type=Application 
MimeType=x-scheme-handler/st-rmm; 
```

Now open `~/.config/mimeapps.list` and make sure it includes these lines:

```ini
[Default Applications] 
x-scheme-handler/st-rmm=st-rmm.desktop 
```

And finally run:

`update-desktop-database ~/.local/share/applications`

Now you should be able to click your normal connect/remote access link in your browser of choice to hop in. 
