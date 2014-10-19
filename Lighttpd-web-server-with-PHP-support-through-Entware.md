We need a usb-flash formatted EXT2 or a usb-hdd formated EXT3, will not work fine on FAT32 or NTFS.

First, setup Entware from [this](https://github.com/RMerl/asuswrt-merlin/wiki/Entware) guide.

Login to router with a terminal like putty and enter this commands:
```
opkg install lighttpd
opkg install php5-cgi
opkg install lighttpd-mod-fastcgi
/opt/etc/init.d/S80lighttpd stop
rm /opt/etc/lighttpd/lighttpd.conf
wget -c -O /opt/etc/lighttpd/lighttpd.conf http://tinyurl.com/amvkxt3 --no-check-certificate
wget -c -O /opt/share/www/index.html http://tinyurl.com/bxfxpq6 --no-check-certificate
wget -c -O /opt/share/www/test.php http://tinyurl.com/b9b34kp --no-check-certificate
/opt/etc/init.d/S80lighttpd start
```
Go to [192.168.1.1:81](http://192.168.1.1:81) and if you see this page, the lighttpd web server is configured correctly

![if](http://i47.tinypic.com/rm5it1.png)

Go to [192.168.1.1:81/test.php](http://192.168.1.1:81/test.php) and if you see this page, the php-mod-fastcgi is configured correctly

![php](http://i50.tinypic.com/i5usfo.png)

###TO ACCESS THE WEBSITE FROM WAN
```
opkg install nano
nano /jffs/scripts/firewall-start
```

Paste this lines
```
#!/bin/sh
iptables -I INPUT -p tcp --destination-port 81 -j ACCEPT
```

Save with **CTRL-O** / **Enter** / and exit nano with **CTRL-X**

Give right permission
```
chmod a+rx /jffs/scripts/firewall-start
```

Go to [192.168.1.1/Advanced_VirtualServer_Content.asp](Port forwarding page) and redirect port 80 to 81, after reboot you should have access from wan.

![wan](http://i47.tinypic.com/309hgqr.png)

Youtube video [here](http://youtu.be/KHABSd7qB2M)

Post issues [here](http://goo.gl/2jisLq)