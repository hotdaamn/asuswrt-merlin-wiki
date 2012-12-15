### About
[Entware](http://code.google.com/p/wl500g-repo/) is a modern alternative to Optware.  Originally designed for OpenWRT, it is also usable by other firmware platforms such as DD-WRT or Tomato.  You can also set this up on your Asuswrt-Merlin based router.

For those unfamiliar with Optware: it's a software repository that offers various software programs that can be installed on your router.  They allow to add new functionalities to your router (provided you have the know-how to properly configure them).  This allows you, for example, to install and run the Asterisk SIP server on your router.

Note that you cannot use both Optware and Entware at the same time.

**Important:** Asus's DownloadMaster is based on Optware, and therefore is NOT conpatible with Entware.  You will have to uninstall DownloadMaster and look at the alternatives provided by Entware.


### Setup

The installation and configuration process must be done through telnet or SSH.  If that part scares you, then forget about Entware already: everything must be installed and configured through telnet/SSH.

First, determine which disk will contain your Entware installation.  This is most likely sda1 if you only have one disk plugged in, with one single partition:

```
cd /mnt/sda1
```

If you have previously used Optware or Download Master you must remove the current installation:

```
rm -rf asusware
reboot
```

Now, let's initialize the Optware support inside Asuswrt:

```
mkdir /mnt/sda1/asusware
touch /mnt/sda1/.asusrouter
```

This will ensure that Asuswrt will properly mount /opt at boot time.

Now that we have the optware directory initialized and automounted by Asuswrt it's time to initialize Entware itself:

```
cd /opt
wget http://wl500g-repo.googlecode.com/svn/ipkg/entware_install.sh
chmod a+rx entware_install.sh
./entware_install.sh
```

Now we have to configure Asuswrt to automatically stop/start services:

```
echo "#!/bin/sh" > /jffs/scripts/services-start
echo "sleep 20" >> /jffs/scripts/services-start
echo "/opt/etc/init.d/rc.unslung start" >> /jffs/scripts/services-start
echo "#!/bin/sh" > /jffs/scripts/services-stop
echo "/opt/etc/init.d/rc.unslung stop" >> /jffs/scripts/services-stop
chmod a+rx /jffs/scripts/*
```
_(Skip the two lines about "#!/bin/sh" if you already have an existing services-xxxx script)_

And you're done.  You might need to reboot your router to fully initialize your Entware environment.


### Usage
Entware's management tool is called "opkg".  Just type "opkg" to see a list of available commands.  the most useful ones would be the following:

```
opkg list
opkg install software_name
opkg remove software_name
```

For example, to install nano, a text editor that is much more user-friendly than vi:

```
opkg install nano
```

