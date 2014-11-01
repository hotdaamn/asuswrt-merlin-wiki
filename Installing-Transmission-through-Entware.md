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

Enter the following content (omit the first line if you already have an existing script)
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
### EMAIL NOTIFICATIONS
If you have a slow internet connection and you want to be notified when a torrent has finished downloading, place the following script which I called _tmail.sh_ in /jffs/scripts but first don't forget to fill: SMTP, FROM, TO, USER and PASS with your credentials.

> WARNING, may become annoying if you are downloading lots of torrents

```
#!/bin/sh

SMTP="your-smtp-server:587"
FROM="your-email-address"
TO="your-email-address"
USER="email-user-name"
PASS="email-password"
FROMNAME="Asus Router"
torrent_name="$TR_TORRENT_NAME"

echo "Subject: Download notification!" >/tmp/tmail.txt
echo "From: \\"$FROMNAME\\"<$FROM>" >>/tmp/tmail.txt
echo "Date: `date -R`" >>/tmp/tmail.txt
echo "" >>/tmp/tmail.txt
echo Transmissionbt has finished downloading "$TR_TORRENT_NAME" on `date +\%d/\%m/\%Y` at `date +\%T` >>/tmp/tmail.txt
echo "" >>/tmp/tmail.txt
echo "Your friendly router." >>/tmp/tmail.txt
echo "" >>/tmp/tmail.txt

cat /tmp/tmail.txt | /usr/sbin/sendmail -S"$SMTP" -f"$FROM" $TO -au"$USER" -ap"$PASS"

rm /tmp/tmail.txt
```
Stop transmission daemon, change this two lines in /opt/etc/transmission/settings.json and start transmission again
```
"script-torrent-done-enabled": true, 
"script-torrent-done-filename": "/jffs/scripts/tmail.sh",
```
### EMAIL IN HTML FORMAT
```
#!/bin/sh

SMTP="your-smtp-server:587"
FROM="your-email-address"
TO="your-email-address"
USER="email-user-name"
PASS="email-password"
FROMNAME="Asus Router"
torrent_name="$TR_TORRENT_NAME"
torrent_version="$TR_APP_VERSION"

logger -t "$0" "Mail sent, about "$TR_TORRENT_NAME""

echo MIME-Version: 1.0 >/tmp/tmail.html
echo Content-Type: text/html >>/tmp/tmail.html
echo "Subject: Download notification" >>/tmp/tmail.html
echo "From: \\"$FROMNAME\\"<$FROM>" >>/tmp/tmail.html
echo "Date: `date -R`" >>/tmp/tmail.html
echo "" >>/tmp/tmail.html
echo "<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">" >>/tmp/tmail.html
echo "<html>" >>/tmp/tmail.html
echo "<head><title></title>" >>/tmp/tmail.html
echo "</head>" >>/tmp/tmail.html

echo "<p>Transmissionbt v"$torrent_version" finished downloading:</p>" >>/tmp/tmail.html
echo "<p><b>"$TR_TORRENT_NAME"<b></p>" >>/tmp/tmail.html
echo "<p>on `date +\%d/\%m/\%Y` at `date +\%T`</p>" >>/tmp/tmail.html
echo "" >>/tmp/tmail.html
echo "<p>Your awesome router.</p>" >>/tmp/tmail.html
echo "<a href="http://tinypic.com?ref=2zod5ja" target="_blank"><img src="http://i40.tinypic.com/2zod5ja.png" border="0" alt="Image and video hosting by TinyPic"></a>" >>/tmp/tmail.html
echo "</body>" >>/tmp/tmail.html
echo "</html>" >>/tmp/tmail.html

cat /tmp/tmail.html | /usr/sbin/sendmail -S"$SMTP" -f"$FROM" $TO -au"$USER" -ap"$PASS"

rm /tmp/tmail.html
```
Example of received email:
![email](http://i40.tinypic.com/2mi2is4.png)
Post issues [here](https://www.asuswrt.eu/transmission-on-asuswrt-routers-through-entware/)