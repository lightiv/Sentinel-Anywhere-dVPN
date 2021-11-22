# Sentinel dVPN
Sentinel dVPN Anywhere Exit Node

## Introduction

This project was created to easily configure and deploy a Sentinel dVPN exit node to any Linux build without use of docker.

## Important Notices:

This installation is derived for the official Sentinel dVPN installation

## Install a fresh installation of Unbuntu Linux

Here is my best practice installation: https://github.com/lightiv/SkyNet/wiki/Ubuntu-Linux-Install-Guide

## Setup

**Login As ROOT**

Setup the core networking files:
```
sudo apt-get install autotools-dev -y && sudo apt-get install libtool-bin -y \ 
&& sudo apt-get install libunbound-dev -y && sudo apt install wireguard -y && curl -y && unzip
```
Get HandShake DNS and the Sentinelnode binary
```
wget https://sentinel-files.s3.filebase.com/hnsd && mv hnsd /usr/bin/hnsd && chmod +x /usr/bin/hnsd && which hnsd
```
```
wget https://sentinel-files.s3.filebase.com/sentinelnode && mv sentinelnode /usr/bin/sentinelnode \
     && chmod +x /usr/bin/sentinelnode && which sentinelnode
```
## Configure for hnsd and clear port 53 for hnsd use
```
nano /etc/hosts
```
Add the following if not found.  The hostname is the results of: *echo $HOSTNAME*
```
127.0.0.1	<hostname>
```
Configure DNS
```
nano /etc/resolv.conf
```
Enter the following if not found.  I only had to change *nameserver 127.0.0.53* to *127.0.0.1*
```
nameserver 127.0.0.1
options edns0 trust-ad
```
Configure nameserver
```
nano /etc/resolvconf.conf
```
Add the following and save
```
name_servers="127.0.0.1"
```
dhcpcd may try to overwrite your resolv.conf to prevent this. The hostname is the results of: *echo $HOSTNAME*
```
nano /etc/dhcpcd.conf
```
Add the following:
```
option dVPN-node
```
NetworkManager has similar behavior to dhcpcd. To prevent it from changing your resolv.conf:
```
nano /etc/NetworkManager/NetworkManager.conf
```
Add the following:
```
[main]
dns=none

[connectivity]
interval=604800
```
