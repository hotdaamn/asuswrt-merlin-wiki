### About
[Entware](http://entware.wl500g.info) is a modern alternative to Optware.  Originally designed for OpenWRT, it is also usable by other firmware platforms such as DD-WRT or Tomato.  You can also set this up on your Asuswrt-Merlin based router.

For those unfamiliar with Optware: it's a software repository that offers various software programs that can be installed on your router.  They allow you to add new functionality to your router (provided you have the know-how to properly configure them).  This allows you, for example, to install and run the Asterisk SIP server on your router.

Note that you cannot use both Optware and Entware at the same time.

**Important:** Asus's DownloadMaster is based on Optware, and therefore is NOT compatible with Entware.  You will have to uninstall DownloadMaster and look at the alternatives provided by Entware.


### Setup

The installation and configuration process must be done through telnet or SSH.  If that part scares you, then forget about Entware already: everything must be installed and configured through telnet/SSH.

###The easy way

Starting with v3.0.0.4.270.25 a new script has been introduced to facilitate Entware installation. After installing a [USB drive](https://github.com/Entware/entware/wiki/USB-Storage-setup), just type in terminal:
```
entware-setup.sh
```

```
admin@RT-AC66U:/tmp/mnt/sda1/asusware# entware-setup.sh
 Info:  This script will guide you through the Entware installation.
 Info:  Script modifies only "entware" folder on the chosen drive,
 Info:  no other data will be touched. Existing installation will be
 Info:  replaced with this one. Also some start scripts will be installed,
 Info:  the old ones will be saved to .entwarejffs_scripts_backup.tgz

 Info:  Looking for available partitions...
[1] --> /tmp/mnt/sda1
 =>  Please enter partition number or 0 to exit
```
Choose a partition where Entware should be installed, in this case is only [1] --> /tmp/mnt/sda1
```
[0-1]: 1
 Info:  /tmp/mnt/sda1 selected.

 Info:  Creating /tmp/mnt/sda1/entware folder...
 * Warning:  Deleting old /tmp/opt symlink...
 Info:  Creating /tmp/opt symlink...
 Info:  Creating /jffs scripts backup...
tar: removing leading '/' from member names
 Info:  Modifying start scripts...
 Info:  Starting Entware deployment....

Connecting to entware.wl500g.info (81.4.122.163:80)
-                    100% |*******************************|  1141   0:00:00 ETA
Info: Checking for prerequisites and creating folders...
Info: Opkg package manager deployment...
Connecting to entware.wl500g.info (81.4.122.163:80)
opkg                 100% |*******************************|   456k  0:00:00 ETA
Connecting to entware.wl500g.info (81.4.122.163:80)
opkg.conf            100% |*******************************|   105   0:00:00 ETA
Info: Basic packages installation...
Downloading http://entware.wl500g.info/binaries/entware/Packages.gz.
Updated list of available packages in /opt/var/opkg-lists/openwrt.
Installing uclibc-opt (0.9.32-4) to root...
Downloading http://entware.wl500g.info/binaries/entware/uclibc-opt_0.9.32-4_entware.ipk.
Installing libc (0.9.32-6) to root...
Downloading http://entware.wl500g.info/binaries/entware/libc_0.9.32-6_entware.ipk.
Installing libgcc (4.6.4-6) to root...
Downloading http://entware.wl500g.info/binaries/entware/libgcc_4.6.4-6_entware.ipk.
Installing libstdcpp (4.6.4-6) to root...
Downloading http://entware.wl500g.info/binaries/entware/libstdcpp_4.6.4-6_entware.ipk.
Installing libpthread (0.9.32-6) to root...
Downloading http://entware.wl500g.info/binaries/entware/libpthread_0.9.32-6_entware.ipk.
Installing librt (0.9.32-6) to root...
Downloading http://entware.wl500g.info/binaries/entware/librt_0.9.32-6_entware.ipk.
Installing ldconfig (0.9.32-6) to root...
Downloading http://entware.wl500g.info/binaries/entware/ldconfig_0.9.32-6_entware.ipk.
Installing findutils (4.5.12-1) to root...
Downloading http://entware.wl500g.info/binaries/entware/findutils_4.5.12-1_entware.ipk.
Configuring ldconfig.
Configuring libgcc.
Configuring libc.
Configuring libpthread.
Configuring libstdcpp.
Configuring librt.
Configuring findutils.
Configuring uclibc-opt.
Updating /opt/etc/ld.so.cache... done.
Info: Cleanup...
Info: Congratulations!
Info: If there are no errors above then Entware successfully initialized.
Info: Found a Bug? Please report at https://github.com/Entware/entware/issues
```
The script will create a new directory named "entware" and not "asusware" like in the "old" way, but the result is the same:
```
admin@RT-AC66U:/tmp/home/root/# cd /opt
admin@RT-AC66U:/tmp/mnt/sda1/entware# 
```
***

### The "old" way
First, determine which disk will contain your Entware installation.  This is most likely sda1 if you only have one disk plugged in, with one single partition:

```
cd /mnt/sda1
```

If you have previously used Optware or Download Master you must remove the current installation:

```
rm -rf asusware[.arm]
reboot
```

Now, let's initialize the Optware support inside Asuswrt:

```
mkdir /mnt/sda1/asusware.arm
touch /mnt/sda1/asusware.arm/.asusrouter
reboot
```

This will ensure that Asuswrt will properly mount /opt at boot time.

After the reboot, the optware directory should be initialized and automounted by Asuswrt.  It's now time to initialize Entware itself:

```shell
wget -O - http://entware.wl500g.info/binaries/entware/installer/entware_install.sh | sh
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
See a youtube video [here](http://www.youtube.com/watch?v=WhlzW_Fl1KA&feature=youtu.be)