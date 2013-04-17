We need a usb-flash formatted EXT2 or a usb-hdd formated EXT3. DON'T TRY TO INSTALL IT ON NTFS OR FAT32 FORMATTED DISKS BECAUSE YOU ONLY GET A BIG HEADACHE!!!

On router UI we have to enable Telnet or SSH under [Administration / System tab](http://192.168.1.1/Advanced_System_Content.asp).

![ssh](http://i45.tinypic.com/6rroqv.png)

To start the optware environment we have to install and remove Download Master right after.

![dm](http://i49.tinypic.com/x585f6.png)

Now login to router with [putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) terminal
```
ipkg update
ipkg upgrade
ipkg install transmission
wget -c -O /opt/etc/init.d/S95transmission http://tinyurl.com/S95transmission
chmod 777 /opt/etc/init.d/S95transmission
/opt/etc/init.d/S95transmission start
/opt/etc/init.d/S95transmission stop
chmod 777 /opt/etc/transmission-daemon/settings.json
rm -r /opt/etc/transmission-daemon/settings.json
wget -c -O /opt/etc/transmission-daemon/settings.json http://tinyurl.com/settings-json
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
wget -c -O /jffs/scripts/firewall-start http://tinyurl.com/firewall-start
chmod a+rx /jffs/scripts/firewall-start
reboot
```
After rebooting type YourWanIp:9091 on preferred internet browser, or install Transmission Remote GUI from [here](https://code.google.com/p/transmisson-remote-gui/)
### EMAIL NOTIFICATIONS
If you have a slow internet connection and you want to be notified when a torrent has finished downloading, place the folowing script witch I called _tmail.sh_ in /jffs/scripts but first don't forget to fill: SMTP, FROM, TO, USER and PASS with your credentials.

> WARNING, may become annoying if you are downloading lots of torrents

```
#!/bin/sh

SMTP="your-smtp-server:25"
FROM="your-email-adress"
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
 
See youtube video [here](http://www.youtube.com/watch?v=hHBEMYJfLi8)

Post issues or feedback [here](http://forums.smallnetbuilder.com/showthread.php?t=8696)