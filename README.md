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
sudo apt-get install autotools-dev -y && sudo apt-get install libtool-bin -y && sudo apt-get install unbound -y && sudo apt-get install libunbound-dev -y && sudo apt install wireguard -y && unzip
```
Get HandShake DNS and the Sentinelnode binary
```
wget https://sentinel-files.s3.filebase.com/hnsd && mv hnsd /usr/local/bin/hnsd && chmod +x /usr/local/bin/hnsd && which hnsd
```
```
wget https://sentinel-files.s3.filebase.com/sentinelnode && mv sentinelnode /usr/local/bin/sentinelnode && chmod +x /usr/local/bin/sentinelnode && which sentinelnode
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
option host_name
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
# Free up port 53 so hnsd will run and not error
```
nano /etc/systemd/resolved.conf
```
Change the following to *no* and uncomment if needed
```
DNSStubListener=no
```
Create a symbolic link for /run/systemd/resolve/resolv.conf with /etc/resolv.conf as the destination
```
ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```
Reboot system and login as **root**
```
reboot -d 0
```
See what is running on port 53:
```
sudo lsof -i :53
```
If port 53 is still in use -> "systemd-r     886 systemd-resolve   13u  IPv4     22076      0t0  TCP 127.0.0.53:53 (LISTEN)"
```
sudo systemctl disable systemd-resolved
```
```
sudo systemctl stop systemd-resolved
```
Stop and disable unbound if using port 53
```
systemctl stop unbound && systemctl disable unbound
```
See what is running on port 53:
```
sudo lsof -i :53
```
# Initialize config.toml and wiregard.toml

```
sentinelnode config init && sentinelnode wireguard config init
```
Edit config.toml to your settings. Change the following in the statement below <NODE_NAME>, <IP_ADDRESS_OF_NODE>, <KEY_NAME>, and <PRICE>.  You can also change the port numbers but they must be the same.
```
sed -i.bak -e 's,^moniker *=.*,moniker = \"<NODE_NAME>\",; s,^listen_on *=.*,listen_on = \"0.0.0.0:43852\",; s,^remote_url *=.*,remote_url = \"https://<IP_ADDRESS_OF_NODE>:43852\",; s,^from *=.*,from = \"<KEY_NAME>\",; s,^price *=.*,price = \"<PRICE>udvpn\",' /root/.sentinelnode/config.toml
```
**Create Keys and fund it.  The key name will be pulled from the config.toml file you just edited.**
```
sentinelnode keys add
```
**Save your addresses and mnemonic!  It is the only way to recover your wallet!**
### Send 100dvpn to the newly created: _operator: sent1..._.  This is needed to pay for node transactions
You can use the command line or Keplr to send the dvpn

# Create Self-Signed Certificates

Change the following in the statement below <"YEARS_AS_DAYS">, <"MONIKER">, <"NAME_FOR_NODE">
```
openssl req -new -newkey ec -pkeyopt ec_paramgen_curve:prime256v1 -x509 -sha256 -days <"YEARS_AS_DAYS"> -nodes -out /root/.sentinelnode/tls.crt -keyout /root/.sentinelnode/tls.key -subj "/C=US/ST=California/L=Los Angeles/O=<"MONIKER">/CN=<"NAME_FOR_NODE">"
```
For example.  This sets the cert expiration to 10 years.:
```
openssl req -new -newkey ec -pkeyopt ec_paramgen_curve:prime256v1 -x509 -sha256 -days 3650 -nodes -out .sentinelnode/tls.crt -keyout .sentinelnode/tls.key -subj "/C=US/ST=California/L=Los Angeles/O=SkyNet | Validators/CN=Sentinel dVPN Node"
```

# Start Sentinel dVPN Anywhere Node
```
sentinelnode start
```
If the above starts without an error.  Stop and create the following service to start and stop the node

Create start script.  You will include your keyring password so no user input is needed
```
nano .sentinelnode/start-dvpn.sh
```
Enter the following:
```
#!/bin/bash

ehco <KEYRING_PASSPHASE> | sentinelnode start
```
```
CTRL+O Enter
```
```
CTRL+x
```
```
chmod +x .sentinelnode/start-dvpn.sh
```
```
sudo nano /lib/systemd/system/dvpn.service
```
Add the following:
```
[Unit]
Description=Sentinel dVPN Node
After=network-online.target

[Service]
User=root
ExecStart=/.sentinelnode/start-dvpn.sh
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```
```
CTRL+O Enter
```
```
CTRL+x
```
```
sudo systemctl enable dvpn
```
```
sudo systemctl start dvpn
```
View the status of your node
```
sudo journalctl --no-hostname -fu dvpn
```
