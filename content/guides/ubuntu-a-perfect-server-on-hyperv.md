+++
authors = ["Jun Wu"]
title = "Set up Ubuntu on Hyper-V - 2024"
date = "2024-06-01"
description = "a guide to use set up a Ubuntu VM on Hyper-V."
tags = [
    "kubernetes"
]
categories = [
    "infra",
    "devops"
]
series = ["ubuntu"]
+++
## Install Ubuntu in Hyper-V

download ubuntu server ISO file from `https://ubuntu.com/download/server`


## Set proxy 

### for console program

```sh
sudo vi .bashrc
```

add the following content

```sh
export http_proxy=http://192.168.137.1:7890/
export https_proxy=http://192.168.137.1:7890/
export ftp_proxy=http://192.168.137.1:7890/
```

### for apt 

```sh
sudo vi /etc/apt/apt.conf
```

add these content

```sh
Acquire::http::proxy "http://192.168.137.1:7890/";
Acquire::https::proxy "http://192.168.137.1:7890/";
Acquire::ftp::proxy "http://192.168.137.1:7890/";
```

### for System-wide

```sh
sudo vi /etc/environment
```

```sh
http_proxy=http://192.168.137.1:7890/
https_proxy=http://192.168.137.1:7890/
ftp_proxy=http://192.168.137.1:7890/
```


## Modify ubuntu archive repo

```sh
sudo vi /etc/apt/sources.list.d/ubuntu.sources
```

then pick one from `https://launchpad.net/ubuntu/+archivemirrors` and replace the old one with the one you chose.

```sh
Types: deb
URIs: http://jp.archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

### Modify hostname and /etc/hosts

```sh
sudo vi /etc/hostname
sudo vi /etc/hosts
```


## Set IPs for Ubuntu server

### Static Internal IP

The static ip is to allow you to access your ubuntu server with a fixed ip every time you access it, because the default switch will assign a random ip to your virtual machine every time you reboot windows, so we need to fix a static ip for your ubuntu server.

### External Switch IP

If your windows computer is connected to a router and the router has dhcp turned on, then you can let your ubuntu server have the same ip as your windows which is assigned by the dhcp function of the router, so it seems that your ubuntu server and your windows are two real machines in the same LAN.