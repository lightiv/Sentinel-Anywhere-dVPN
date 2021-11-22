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
wget https://sentinel-files.s3.filebase.com/sentinelnode && mv sentinelnode /usr/bin/sentinelnode && chmod +x /usr/bin/sentinelnode && which sentinelnode
```
