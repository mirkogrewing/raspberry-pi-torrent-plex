# Setting Up PureVPN on Raspberry Pi

## Requirements

1. A working internet connection
2. A Premium PureVPN account

## Procedure

### Installation

The solution is based on OpenVPN, therefore let's proceed with installing it if it is not already:

```
sudo apt-get install openvpn
```

### Basic Configuration

Now we can download the configuration files created by PureVPN team:

```
wget https://s3-us-west-1.amazonaws.com/heartbleed/linux/linux-files.zip
```

Unzip them:

```
unzip linux-files.zip
```

A new directory called "Linux OpenVPN Updated files" is created. We rename it for ease, and we move to this new directory:

```
mv "Linux OpenVPN Updated files" PureVPN
cd PureVPN
```

Now we can move all the configurations files where OpenVPN will be looking for them, and move to that directory:

```
sudo cp ca.crt TCP/* UDP/* Wdc.key /etc/openvpn/
cd /etc/openvpn/
```

We can now test the connection by running:

```
sudo openvpn Luxembourg1-tcp.ovpn
```

Please note that you can potentially use any of the configuration files present in the folder.

PureVPN will request your username and password and will confirm a successful connection. You can interrupt the connection by simply pressing `CTRL+C`.

If you get a "TLS handshake failed" message, try and switch to an UDP.

Once you can connect successfully you can store your log in information so that PureVPN doesn't request the user to type them at every connection. To do so, run the following command from `/etc/openvpn/`:

```
sudo sed -i "s/auth-user-pass/auth-user-pass pass.txt/g" *.ovpn
```

so that the file with the log-in information will be automatically appended every time a .ovpn file is launched. We can now create the file, in the same directory:

```
sudo nano pass.txt
```

Now add your username on the first line, your password on the second, being careful not to leave any extra space at the end of the line, or any extra empty line.

You can now launch the VPN connection again and verify that it connects without requesting the log-in information:

```
sudo openvpn Luxembourg1-tcp.ovpn
```

#### Prepare a configuration file

OpenVPN can only run as a daemon with a .conf file, not .ovpn. Therefore, we need to pick one .ovpn file and change the extension to .conf:

```
sudo cp Luxembourg1-tcp.ovpn purevpn.conf
```

### Execute custom commands on start/stop

Once the connection has been established, we need to run a script provided by OpenVPN that will adjust the DNS settings.

Also, you might want to pause torrent download when the VPN comes down, and resume it again once the VPN is up. To do so we can utilise an internal feature of OpenVPN that allows to run scripts when the connection comes up or goes down.

So we start from creating the scripts that will be executed when the connection is established or closed:

```
$ sudo touch /etc/openvpn/up.sh
$ sudo touch /etc/openvpn/down.sh
```

and we make sure that the scripts are called, by adding the following lines at the very bottom of `/etc/openvpn/purevpn.conf`:

```
script-security 2
up /etc/openvpn/up.sh
down /etc/openvpn/down.sh
```


#### Update DNS settings

OpenVPN provides a script to update settings in `/etc/resolv.conf` when the VPN is established. The script is `/etc/openvpn/update-resolv-conf` and we have to run it first thing when the connection is established, and the last, when the connection is closed.

#### Pause Transmission if VPN is down

If you don't want torrent downloading to be active while the VPN is down, you have to pause your bittorrent client. Transmission allows to pause downloads with:

```
$ transmission-remote --auth transmission:transmission --torrent all --stop
```

and resume them with:

```
$ transmission-remote --auth transmission:transmission --torrent all --start
```

#### Putting things together

The final content of `/etc/openvpn/up.sh` will be:

```
#!/bin/sh

echo "Updating DNS Settings"
/bin/bash /etc/openvpn/update-resolv-conf
echo "Starting Transmission Torrent Downloading"
transmission-remote --auth transmission:transmission --torrent all --start
```

while `/etc/openvpn/up.sh` will be:

```
#!/bin/sh

echo "Stopping Transmission Torrent Downloading"
transmission-remote --auth transmission:transmission --torrent all --stop
echo "Restoring DNS Settings"
/bin/bash /etc/openvpn/update-resolv-conf
```

### Launching at Boot

To have your VPN established at boot, we have to specify which config file OpenVPN shall use:

```
$ sudo vim /etc/default/openvpn
```

and change the parameter `AUTOSTART` to match the name of your config file (without .conf). We previously renamed the initial configuration file provided by PureVPN as `purevpn.conf`, therefore:


```
AUTOSTART="purevpn"
```


# To-Do
...
