## Introduction
Asus's DownloadMaster suffers from various issues: old versions of Transmission and OpenSSL, no way to disable aMule which will hog all your bandwidth, etc...

I recommend uninstalling Download Master, and manually setting up Transmission through Entware instead.  This will bring various benefits:

1. Better performance overall
2. Magnet link support
3. Most people don't care about aMule or NZBGet - that will cut through the fat.

## Initial configuration
For this I will assume your disk is mounted as /mnt/sda1/ (just adjust the paths as needed if yours is mounted as /mnt/sdb1 instead, for instance).

I will also assume that your disk is already formatted as either Ext2 or Ext3.  If not, look on the web for information on how to reformat your disk.

Initial steps:

* Enable the JFFS partition (as explained [here](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS))
* Setup Entware (as explained [here](https://github.com/RMerl/asuswrt-merlin/wiki/Entware))

Install the nano editor (unless you are already comfortable with the vi editor):

```
opkg install nano
```

## Installation
We need to install Transmission:

```
opkg install transmission-daemon
opkg install transmission-web
```

Create the data directories (adjust as desired):

```
mkdir /mnt/sda1/Torrent/
mkdir /mnt/sda1/Torrent/Incomplete
mkdir /mnt/sda1/Torrent/Watch
mkdir /mnt/sda1/Torrent/Completed
```

Make sure Transmission isn't already running, then edit its configuration:

```
/opt/etc/init.d/S88transmission stop
nano -w /opt/etc/transmission/settings.json
```
 
You will want to adjust the following paths:

```
"download-dir": "/mnt/sda1/Torrent/Completed",
"watch-dir": "/mnt/sda1/Torrent/Watch",
"incomplete-dir": "/mnt/sda1/Torrent/Incomplete",
```

It's also recommended to password-protect the webui.  Set the following parameters:

```
"rpc-authentication-required": true,
"rpc-username": "admin",
"rpc-password": "yourpassword",
```
Your password will be hashed the first time Transmission runs, so it's safe to enter it as clear text there.

## Firewall configuration
We need to create a user script that will open the required port in the firewall.  If you changed this from the default value in settings.json then update this accordingly.

```
nano -w /jffs/scripts/firewall-start
```

Enter the following content (ommit the first line if you already have an existing script)
```
#!/bin/sh
iptables -I INPUT -p tcp --destination-port 51413 -j ACCEPT
iptables -I INPUT -p udp --destination-port 51413 -j ACCEPT
```

Then make it executable:

```
chmod a+rx /jffs/scripts/firewall-start
```

## Using it
Everything is now configured.  You can manually start it immediately (it will automatically start at boot time):

```
/jffs/scripts/firewall-start
/opt/etc/init.d/S88transmission start
```

Access it through http://192.168.1.1:9091/transmission 

See youtube video [here](http://www.youtube.com/watch?v=WhlzW_Fl1KA&feature=youtu.be)
