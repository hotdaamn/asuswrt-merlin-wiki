We need a usb-flash formatted EXT2 or a usb-hdd formated EXT3. DON'T TRY TO INSTALL IT ON NTFS OR FAT32 FORMATTED DISKS BECAUSE YOU ONLY GET A BIG HEADACHE!!!

On router UI we have to enable Telnet or SSH under [Administration / System tab](http://192.168.1.1/Advanced_System_Content.asp).

![ssh](http://i45.tinypic.com/6rroqv.png)

To start the optware environment we have to install and remove Download Master right after.

![dm](http://i49.tinypic.com/x585f6.png)

Now login to router with [putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) terminal
```
ipkg update
ipkg upgrade
ipkg install transmission libiconv wget-ssl
/opt/bin/wget -c -O /opt/etc/init.d/S95transmission http://goo.gl/VCgvw2 --no-check-certificate
chmod 777 /opt/etc/init.d/S95transmission
/opt/etc/init.d/S95transmission start
/opt/etc/init.d/S95transmission stop
chmod 777 /opt/etc/transmission-daemon/settings.json
rm -r /opt/etc/transmission-daemon/settings.json
/opt/bin/wget -c -O /opt/etc/transmission-daemon/settings.json http://goo.gl/PIHW2f --no-check-certificate
app_set_enabled.sh transmission yes
```

Go to [192.168.1.1:9091](http://192.168.1.1:9091) on preferred internet browser (by default username: **admin** and pass: **admin**)

The default download folder is on /mnt/sda1/Transmission (will be created automatically with the first downloaded torrent and can be changed editing settings.json file, for this we need nano editor:
```
ipkg install nano
```
_Be sure transmission-daemon is not running when editing settings.json or it will be overwritten_
```
/opt/etc/init.d/S95transmission stop
nano /opt/etc/transmission-daemon/settings.json
```
Here you can edit any settings like enable/change username, password and save with CTRL-O and ENTER, to exit nano type CTRL-X

Start transmission-daemon again
```
/opt/etc/init.d/S95transmission start
```
### WAN ACCESS
If you want to access transmission from WAN like work, school, smartphone, tablet or some other device we need to open the port 9091 but the firmware doesn't allow port forwarding to the router himself, for that we will use the scripts on /jffs partition

On router UI we have to enable "jffs partition" and "Format JFFS partition at next boot" under [Administration / System tab](http://192.168.1.1/Advanced_System_Content.asp) and reboot router

![jffs](http://i49.tinypic.com/x3ehpc.png)
```
/opt/bin/wget -c -O /jffs/scripts/firewall-start http://goo.gl/ZFHdHC --no-check-certificate
chmod a+rx /jffs/scripts/firewall-start
reboot
```
After rebooting type YourWanIp:9091 on preferred internet browser, or install Transmission Remote GUI from [here](https://code.google.com/p/transmisson-remote-gui/)
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
Stop transmission daemon, change this two lines in /opt/etc/transmission-daemon/settings.json and start transmission again
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
See youtube video [here](http://www.youtube.com/watch?v=hHBEMYJfLi8)

Post issues or feedback [here](http://goo.gl/S2QQ6W)