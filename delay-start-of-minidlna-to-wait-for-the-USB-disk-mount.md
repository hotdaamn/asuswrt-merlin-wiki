when the USB disk is getting to slow mounted, you can use the following script to delay the minidlna start do not screw the mount points. 

```
#!/bin/bash
sleep 2m
# the script will wait for 2 Minutes, than all the USB Disks should be ready.....
datestr=$(date +%Y%m%d_%H%M%S)
# date used to add to the log file (append) so all the logs will be collected with the date and time no log overwriting....

# Variables you need to change for your needs:
ps=/jffs/ps
# just using the jffs to export the running processes and get the result to do the next steps....

mountpoint=/tmp/mnt/DISK1
# the disk you use for media server

dlna_config=/tmp/mnt/DISK1/Scripts/minidlna.conf
# the configuration file created for minidlna to use the custom settings.

#--------------DO NOT CHANGE ----------------------
if grep $mountpoint /etc/mtab &>/dev/null; then
echo $datestr "USB disk for media server operation is mounted" >> /tmp/syslog.log
else exit
fi
#when the script can not fine the the USB disk or it is not mounted yet, than the action will be Exit and the script stops..... otherwise we will go further.....

/bin/ps > $ps
if
grep minidlna /jffs/ps &>/dev/null; then
echo $datestr "minidlna server is already started and is not using its new configuration" >> /tmp/syslog.log
else
/usr/sbin/minidlna -f $dlna_config -R
fi
rm $ps
```

the minidlna.conf I took from the original provided from Asus like in example this:

```
`network_interface=br0
port=8200
friendly_name=RT-AC56U
db_dir=/tmp/mnt/DISK1/Router
enable_tivo=no
strict_dlna=no
presentation_url=http://192.168.102.1:80/
inotify=yes
notify_interval=600
album_art_names=Cover.jpg/cover.jpg/Thumb.jpg/thumb.jpg
media_dir=/tmp/mnt/DISK1/Media
serial=d8:50:e6:50:09:c0
model_number=3.0.0.4.374.38
```

so the script will load this configuration file and load minidlna with the above settings. 

**IMPORTANT: you need to disable the Media Server in the GUI interface to work proper. **

For the automatic start you need to add the following line to /jffs/scripts/post-mount

```
sh /tmp/mnt/DISK1/Scripts/mini_dlna.conf & 
```

When the disk will still not be ready than you can add this to cron to run every X amount of minutes / hours.