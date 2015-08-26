when the USB disk is getting to slow mounted, you can use the following script to delay the minidlna start do not screw the mount points. Put following content to `/jffs/scripts/post-mount`:

```
#!/bin/sh

mountpoint=/tmp/mnt/DISK1
# the disk you use for media server

dlna_config=/tmp/mnt/DISK1/Scripts/minidlna.conf
# the configuration file created for minidlna to use the custom settings.


if [ $1 = $mountpoint ]
then
  /usr/sbin/minidlna -f $dlna_config -R
fi
```

The `minidlna.conf` I took from the original provided from Asus like in example this:

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

Don't forget to make script executable:

```
chmod +x /jffs/scripts/post-mount
```

Reboot router to take effect. 