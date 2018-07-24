# Torrent Machine and Plex Media Server on a Raspberry Pi

This project is a collection of guidelines and tutorials explaining how to create a fully autonomous torrent machine and Plex Media Server out of a Raspberry Pi.

## Requirements

1. A Raspberry Pi with [Raspbian](https://www.raspberrypi.org/downloads/raspbian/), connected to the internet, whose intranet IP is known - the installation of Raspbian is out of scope.
1. An external USB Drive.
1. A private VPN account.

For this tutorial I used:
1. 1 Raspberry Pi 3 - Model B
1. 1 Trascend StoreJet 25M3 (2 TB) formatted with ExFAT
1. 1 [PureVPN](https://www.purevpn.com) premium account

## Pre-requisites

### Prepare your Raspberry Pi

Make sure that your Raspberry Pi has the latest updates installed:

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

### Connect the external USB drive

I referred to the following table to determine the best suited filesystem for my network:

File System|Windows XP|Windows 7/8/10|macOS (10.6.4 and earlier)|macOS (10.6.5 and later)|Ubuntu Linux|Playstation 4|Xbox 360/One
------------ | ------------- | ------------ | ------------- | ------------ | ------------- | ------------ | -------------
NTFS |Yes|Yes|Read Only|Read Only|Yes|No|No/Yes
FAT32|Yes|Yes|Yes|Yes|Yes|Yes|Yes/Yes
exFAT|Yes|Yes|No|Yes|Yes (with ExFAT packages)|Yes (with MBR, not GUID)|No/Yes
HFS+|No|(read-only with Boot Camp)|Yes|Yes|Yes|No|Yes
APFS|No|No|No|Yes (macOS 10.13 or greater)|No|No|No
EXT 2, 3, 4|No|Yes (with third-party software)|No|No|Yes|No|Yes

I decided to use ExFAT since I only work with Mac OS and Linux.

1. Connect an ExFAT USB Drive

### Set up the VPN

Make sure that the Raspberry Pi connects to your preferred VPN on start-up.

1. Configure PureVPN

### Install Transmission BitTorrent

Execute the following command:

```
$ sudo apt-get install -y transmission-daemon transmission-cli transmission-common
```

And stop the demon to apply changes to the configuration:

```
$ sudo service transmission-daemon stop
$ sudo vi /etc/transmission-daemon/settings.json
```

IN PROGRESS

# To-Does
- [ ] Document PureVPN command line configuration
- [ ] Document ExFAT command line configuration
- [ ] Open a pull request
