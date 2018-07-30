# Torrent Machine and Plex Media Server on a Raspberry Pi

This project is a collection of guidelines and tutorials explaining how to create a fully autonomous torrent machine and Plex Media Server out of a Raspberry Pi.

## General Notes

### Operative System

I execute the following commands on a new installation of Raspbian 9. As an editor I use vim, that I installed after the first boot with:

```
$ sudo apt-get install vim-nox
```

### Hardware

I used for this tutorial a Raspberry Pi 3B. Using a fresh Raspbian install and default settings I could not get the device to work for more than 20 minutes consecutively without freezing.

The Raspberry Pi 3 suffers apparently from problems due to voltage and heating. Therefore I tweaked settings and it stopped hanging.

1. [Settings for Raspberry Pi 3 - Model B](settings/raspi-3b.md)

## Requirements

1. A Raspberry Pi with [Raspbian](https://www.raspberrypi.org/downloads/raspbian/), connected to the internet, whose intranet IP is known - the installation of Raspbian is out of scope.
1. An external USB Drive.
1. A private VPN account.

For this tutorial I used:

1. 1 Raspberry Pi 3 - Model B (Linux raspberrypi 4.14.52-v7+ & Raspbian 9)
1. Static IP address for the Raspberry Pi (192.168.0.10)
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

1. [Connect an ExFAT USB Drive](extra/external-drives/ExFAT.md)

## Transmission BitTorrent

### Set-Up and Configuration

Execute the following command:

```
$ sudo apt-get install -y transmission-daemon transmission-cli transmission-common
```

And stop the demon to apply changes to the configuration that will allow you to manage Transmission remotely:

```
$ sudo service transmission-daemon stop
$ sudo vi /etc/transmission-daemon/settings.json
```

Look for `rpc-whitelist` and `rpc-whitelist-enabled` and make sure they look like this:

```
"rpc-whitelist": "127.0.0.1,192.168.*.*",
"rpc-whitelist-enable": "true",
```

Please note that this is assuming that your internal network works on 192.16.0.1/24. If otherwise please change the value accordingly.

Also, you can modify `download-dir` and `incomplete-dir` to point at the external USB drive, for example:

```
"download-dir": "/media/storage/downloads",
"incomplete-dir": "/media/storage/.tmp",
```

Please make sure that the directories exists on your external USB drive.

Once you are done you can restart the Transmission service:

```
$ sudo service transmission-daemon start
```

### Set-Up the Web Interface

The web interface is already available at the address `http://192.168.0.10:9091` (replace the IP address with the one used by the Raspberry Pi in your network).

You can login using `transmission` as both username and password.

### OPTIONAL: Set-Up a VPN

Make sure that the Raspberry Pi connects to your preferred VPN on start-up.

1. [Configure PureVPN](extra/vpn/purevpn.md)

### OPTIONAL: Set-Up RSS Support

If you want to automate even further your flow, by having Transmission downloading your favourite videos via an RSS feed, you can install Transmission-RSS.

#### Dependencies

The easiest is to install Transmission-RSS via gem. We will need a few packages to do so:

```
$ sudo apt-get install build-essential patch ruby-dev zlib1g-dev liblzma-dev
```

We also need gem's bundler:

```
$ sudo gem install bundler
```

#### Installation

Now we download the latest version of Transmission-RSS:

```
$ git clone https://github.com/nning/transmission-rss

```

We enter the main directoy, we bundle it, and we build:

```
$ cd transmission-rss/
$ bundle
$ sudo gem build transmission-rss.gemspec
```

and finally, we install it:

```
$ sudo gem install transmission-rss-*.gem
```

#### Configuration

According to the README, I was supposed to find a configuration file (`/etc/transmission-rss.conf`), but it didn't exist. So I created it and put some basic  parameters:

```
feeds:
        - url: http://link_to_my_rss_feed

update_interval: 600

add_paused: false

server:
        host: localhost
        port: 9091
        rpc_path: /transmission/rpc

login:
        username: transmission
        password: transmission

log:
        target: /var/log/transmissiond-rss.log
        level: debug

privileges:
        user: root
        group: root

client:
        timeout: 5

fork: false

pid_file: false

seen_file: ~/.config/transmission/seen
```

Please note that **the config file cannot contain any TAB**, but only spaces. Therefore, if your editor is automatically indenting sub-options please verify that indentations only contain spaces. Any TAB will prevent Transmission-RSS from launching, throwing an error (e.g. "found character that cannot start any token").

#### Launching at Startup

To have Transmission-RSS starting at boot, we can create a service. Create `/etc/systemd/system/transmission-rss.service` and paste the following inside:

```
[Unit]
Description=Transmission RSS daemon.
After=network.target transmission-daemon.service

[Service]
Type=forking
ExecStart=/usr/local/bin/transmission-rss -f
ExecReload=/bin/kill -s HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

To have the service launched at boot:

```
$ sudo systemctl enable transmission-rss
```

### Check

If you reboot now, you should have:

1. Your VPN (if configured) via OpenVPN
2. Transmission
3. Transmission-RSS

launching at boot. Running the command:

```
$ ps xuaw | grep 'openvpn\|transmission'
```

You should see the three running:

```
root       401  5.1  0.5   8640  5388 ?        Ss   15:00   1:45 /usr/sbin/openvpn --daemon ovpn-purevpn --status /run/openvpn/purevpn.status 10 --cd /etc/openvpn --config /etc/openvp /purevpn.conf --writepid /run/openvpn/purevpn.pid
debian-+   536  4.0  2.1 101268 20424 ?        Ssl  15:00   1:21 /usr/bin/transmission-daemon -f --log-error
root       563  0.0  2.2  40436 21596 ?        Sl   15:00   0:00 /usr/bin/ruby2.3 /usr/local/bin/transmission-rss -f
```

## Plex Media Server

### Installation

To install Plex Media Server we need to add a new repository and it's key:

```
$ wget -O - https://dev2day.de/pms/dev2day-pms.gpg.key | sudo apt-key add -
$ echo "deb https://dev2day.de/pms/ stretch main" | sudo tee /etc/apt/sources.list.d/pms.list
```

We can now update our package list and install Plex Media Server:

```
$ sudo apt-get update
$ sudo apt-get install plexmediaserver-installer
```

### Configuration

The configuration is stored in `/etc/default/plexmediaserver.prev`

We need to change the user the server run with from `plex` to the user that you are logging in with (`pi` by default):

```
PLEX_MEDIA_SERVER_USER=pi
```

and we can reboot:

```
$ sudo reboot
```

### Initial Setup

You can complete the setup opening the following link with your browser:

```
http://192.168.0.10:32400/web/
```

Don't forget to replace the IP with the one that your Raspberry Pi is using in your network.

When you are requested to add libraries, you should add the download folder of Transmission, in our example `/media/storage/downloads`.
