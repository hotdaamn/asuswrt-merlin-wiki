This tutorial is based on the "Installing Transmission through Entware" tutorial by TeHashX. I have tested it on my AC68U, but assume it will work just as well on other Entware enabled ARM routers running Asuswrt-Merlin.


## Introduction
Deluge is a BitTorrent client written in Python. It is free, open source software and aims to be a lightweight, secure, feature-rich client. 

## Initial configuration
For this, I will assume your disk is mounted as /mnt/sda1/ (just adjust the paths as needed if yours is mounted as /mnt/sdb1 instead, for instance).

I will also assume that your disk is already formatted as either Ext2 or Ext3.  If not, look on the web for information on how to reformat your disk.

Initial steps:

* Enable the JFFS partition (as explained [here](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS))
* Setup Entware (as explained [here](https://github.com/RMerl/asuswrt-merlin/wiki/Entware))

Install the nano editor (unless you are already comfortable with the vi editor):

```
opkg install nano
```

## Installation
Install Deluge:

```
opkg install deluge
opkg install deluge-ui-web
```

Create the data directory:

_Note: You will have to add the data directory to the Deluge configuration (discussed below). If you wish, you may create separate directories for the "Download to", "Move completed to, "Copy of .torrent files to", "Autoadd .torrent files from" options and update the configuration accordingly._

```
mkdir /mnt/sda1/deluge/
```

Start Deluge and its Web UI:

```
/opt/etc/init.d/S80deluged start
/opt/etc/init.d/S81deluge-web start
```

Access the Web UI at port 888 (default password is "deluge"):

```
http://www.asusrouter.com:888/
```
OR
```
http://<Your Router IP>:888/
```

You can now configure Deluge as desired, either through the web interface or using nano as follows (recommended that you stop Deluge prior to making the config changes through the terminal):

```
/opt/etc/init.d/S80deluged stop
nano /opt/etc/deluge/core.conf
```

In line with the discussion earlier, you may edit the various folder options through the Web UI or the core.conf file. The change to the download location within core.conf has to be made at the following line:
```
"download_location": "/mnt/sda1/deluge/",
```

Other folder options include (change if desired):
```
"move_completed_path": "/root/Downloads",
"torrentfiles_location": "/root/Downloads", 
"autoadd_location": "/root/Downloads",
```

By default, Deluge is set to use a random port. You may wish to assign a static port to Deluge either through the Web UI or through the core.conf file as follows:

_Note: xxxxx represents the port number_

For outgoing:
```
"random_outgoing_ports": false, 
"outgoing_ports": [
    xxxxx, 
    xxxxx
  ],
```

For incoming:
```
 "random_port": false, 
 "listen_ports": [
    xxxxx, 
    xxxxx
  ], 
```

Open the port(s) in the firewall as follows:

```
nano -w /jffs/scripts/firewall-start
```
Enter the following content (omit the first line if you already have an existing script):
```
#!/bin/sh
iptables -I INPUT -p tcp --destination-port xxxxx -j ACCEPT
iptables -I INPUT -p udp --destination-port xxxxx -j ACCEPT
```

Make it executable:

```
chmod a+rx /jffs/scripts/firewall-start
```

## Using it
You may now start the firewall and deluge scripts so that the changes take effect immediately. Otherwise, they will be auto started during the next boot-up.

```
/jffs/scripts/firewall-start
/opt/etc/init.d/S80deluged start
```

If Deluge doesn't auto start after boot up, then edit the post-mount script:

```
nano /jffs/scripts/post-mount
```

Add the following line (change the time in seconds as necessary):

```
sleep 10
/opt/etc/init.d/rc.unslung restart
```