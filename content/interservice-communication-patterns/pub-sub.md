---
title: "Pub-Sub Model"
date: 2023-10-03T15:26:15Z
lastmod: 2018-12-08T15:26:15Z
publishdate: 2018-11-23T15:26:15Z
draft: false
weight: 6
description: InterserviceCommunicationPatterns
TableOfContents: true
---

# Basis of Wifi Network Security
## Media Access Control (MAC) Address
```shell
ifconfig
```
you may see:
- eth0: A physical network card used for LAN Network used in wired connection. 
- wlan0: A physical network adapter used for wireless connection.  
- lo: loopback virtual network interface which lets you use the localhost. This will not have a MAC address.

```Note:``` You may see different names for the interface names. The following assumes the name of wireless interface as wlan0 for simplicity & clarity.
## Changing the MAC address
To be able to do to this following are the steps:
1. Disable the interface
```shell
ifconfig wlan0 down
```
2. Set the Hardware address (has to start with 00)
```shell
ifconfig wlan0 hw ether 00:11:22:33:44:55
```
3. Start the interface
```shell
ifconfig up
```
Once restarted the MAC address will be restored. 

```Note:``` It's possible that the Network Manager resets the MAC address. In that case do the following:

## If MAC Address get Auto-Reset
```shell
vim /etc/NetworkManager/NetworkManager.conf 
```
Add the following lines
```text
[connection]
ethernet.cloned-mac-address=preserve
wifi.cloned-mac-address=preserve
```
restart the network manager.
```shell
service network-manager restart
```

## Wireless LAN Managed v/s Monitor mode.
1. See the current Mode of the wireless device
```shell
iwconfig
```
this will show Mode: Managed for wireless Network. Following are the steps
2. Disable the wireless LAN interface
```shell
ifconfig wlan0 down
```
3. Kill the network manager process which will interfere with Monitor mode.
```shell
airmon-ng check kill
```
4. Enable Monitor Mode
```shell
airmon-ng start wlan0
```
```OR```
```shell
iwconfig wlan0 mode monitor
```
5. Bring up the wireless interface.
```shell
ifconfig wlan0 up
```

## Packet Sniffing
1. While on the Monitor mode, then
```shell
airodump-ng wlan0
```
important attributes:
* BSSID - MAC Address of target Network
* ESSID - Network name
* PWR - Signal Strength
* Beacons - Frames sent by network, broadcast the address.
* CH - Channel that the network works on.
* MB - Maximum Speed
* ENC - Encryption
* CIPHER -
* Auth - 

## Bands of Wifi Network
Two - 2.4 & 5 Hz
```shell
airodump-ng --band abg wlan0 
```

## Sniffing
```shell
BSSID=00:11:22:33:44:55
CHANNEL=2
airodump-ng --bssid $BSSID --channel $CHANNEL --write test-file wlan0
```
Now you see two sections when sniffing a particular target network. The second section shows the stations that being connected to.
