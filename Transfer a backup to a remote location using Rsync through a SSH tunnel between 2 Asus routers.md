***
> ### Before going further, please note that you may prefer (if it works) use a similar built-in feature:
> ![Builtin](https://cloud.githubusercontent.com/assets/3483165/5582361/bf776b74-9036-11e4-9844-85912b2993a0.png)
***

###  That said, in the following tutorial, we will learn how to setup the transfer of a backup to a remote site using a ssh channel between 2 ASUS routers with Rsync as transporter.

> Context: 

> * on a site (1080), someone makes a daily backup of all his PCs important files on the local router usb disk .
> * on another site (8075), someone else makes a daily backup of all his PCs important fileson the local router usb disk.
> * each of them wants to place a copy of its local backup to a remote site, meaning that each of them wants to daily transfer its backup to the other router usb disk.
> * when possible, all commands come from 1080.

> If you want something different, please adapt the rsync commands and the scripts to your needs.

 [Rsync](http://en.wikipedia.org/wiki/Rsync) is a "classic" linux program used to synchronize/copy efficiently data between two locations. 

> "[Rsync](https://rsync.samba.org/ftp/rsync/rsync.html) is a fast and extraordinarily versatile file copying tool. It can copy locally, to/from another host over any remote shell, or to/from a remote rsync daemon. It offers a large number of options that control every aspect of its behavior and permit very flexible specification of the set of files to be copied. It is famous for its delta-transfer algorithm, which reduces the amount of data sent over the network by sending only the differences between the source files and the existing files in the destination. Rsync is widely used for backups and mirroring and as an improved copy command for everyday use.

> Rsync finds files that need to be transferred using a "quick check" algorithm (by default) that looks for files that have changed in size or in last-modified time. Any changes in the other preserved attributes (as requested by options) are made on the destination file directly when the quick check indicates that the file's data does not need to be updated."

The source and the destination could be "local", even on the same server/router, but our goal being to secure a backup, we will do it to a remote location. We will also secure the transfer itself by using [SSH ](http://en.wikipedia.org/wiki/Secure_Shell). Secure Shell (SSH) is a cryptographic network protocol for secure data communication, remote command-line login, remote command execution, etc. To make it even more secure, we will use a private/public key schema (instead of id/password).

In my specific case, the routers are named:
* RT-1080 (the local)
* RT-8075 (the remote)

Both need Rsync installed. RT-1080 is the "master", i.e. the one responsible for all actions going on between the 2 routers, and RT-8075 is the "slave", just reacting to RT-1080 requests. In my specific case, each router has a 4TB USB3 disk (on the USB3 port, indeed), and each disk has 2 partitions: 
* the "data partition" (~4TB)
* the "router partition" (~5GB)

> **Please note that you don't absolutely need the "router partition", and for the purpose of this tutorial, we will consider that there is just one disk with one partition**

To **Rsync**, you will possibly need a [swap space](http://en.wikipedia.org/w/index.php?title=Paging) because the router memory could be too limited for the process, depending on the size of the backup. If required, the swap space could be set on a file located on the router partition, or more simply on the main disk, and Optware/rsync could also be installed on that partition. As mentioned, for this tutorial everything will be installed on the main partition. 

Simply said, [Optware](http://en.wikipedia.org/wiki/Optware) is a mechanism that simplifies the installation of some programs on the router.

The data partition will contain:
* the local backup: bckp-1080
* the remote backup: bckp-8075
* the backup logs: bckp-logs

In my specific case, [Windows File History](http://www.pcmag.com/article2/0,2817,2418904,00.asp) is the tool used to backup two local PC on RT-1080, and same thing on the remote side, on RT-8075.  The goal was therefore to send bckp-1080 to RT-8075, and bckp-8075 to RT-1080. The backups are scheduled to run daily at night using CRON. [CRON](http://en.wikipedia.org/wiki/Cron) is the standard Linux tool used to schedule processes on a periodic way.
The CRU command will be used to setup the CRON job:
> CRU= Cron Utility

> add:    cru a <unique id> <"min hour day month week command">

> delete: cru d <unique id>

> list:   cru l

I also use:
* **JFFS**, 
* **fstab**, one of the [Custom config files](https://github.com/RMerl/asuswrt-merlin/wiki/Custom-config-files) and 
* the ["User scripts"](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) of RMerlin firmware version. Two scripts will be setup:
 * **init-start**
 * **post-mount**

[JFFS](http://en.wikipedia.org/wiki/JFFS) is a [writable section of the flash memory](https://github.com/RMerl/asuswrt-merlin/wiki/Jffs) used to store some user's data/program/scripts. Intrinsically, the router is a device with no permanent memory, memory being erased on each shutdown, with the exception of the JFFS space. Please note that the JFFS space has to be also backup because **it could be** overwritten by a new firmware version.

[Fstab](http://en.wikipedia.org/wiki/Fstab) is a linux configuration file used to mount the disks. This file will be stored in the folder **Config** located under **JFFS**. The User's Scripts are event-triggered and will be used to initiate some actions on the router reboot, and on the post-mount of the disk/partitions. The **User's scripts** are located under the folder "**scripts**" also under **JFFS**. Therefore, when backing up the JFFS space, at the same time I backup **fstab** and the **scripts**.

I will go first with the most simplest approach:
* using the backup disk as "system disk", meaning that we will use the usb backup disk to install all the required elements.
* not creating the swap space (because it could be not required if your backup is not "that big").

***
### Procedure:
***
**On RT-1080 (local router):**
* Connect a USB disk (ideally usb 3) on one of the usb port (ideally the usb 3 port if you use a usb3 disk...)
* Select "USB Applications" on the main menu of the GUI.
* Select "Media Services and Servers" on the new page
* Select the tab labeled: "Network Place"
* Turn on "Enable Share" (4)
* For sake of simplicity at this moment, select "Allow guest login".(5) You can change it later and give specific rights to specific persons)
* Enter your specific "Work group" (6)
* Apply the changes (7)
   ![CreateShares1](https://cloud.githubusercontent.com/assets/3483165/5582437/aa41c190-9037-11e4-97c4-415e61fc668d.png)
   ![a](https://cloud.githubusercontent.com/assets/3483165/5597070/ff25c5ce-9269-11e4-890a-d30331138de6.png)

That done, on the same page, on the bottom part:
* Select the main disk (1)
* Create a share/folder "bckp-1080" for the local backup (2)
* Create a share/folder "bckp-8075" for storing the remote backup (2)
* Create a share/folder "bckp-logs for storing the logs of the backup processes (2)
* Optionally create a share/folder "Router" for the route/system needs (2)
* Apply the changes (3)
   ![a](https://cloud.githubusercontent.com/assets/3483165/5597134/222f2078-926b-11e4-9214-b6a7c0afac9b.png)

Enable ssh and jffs and format jffs
* Select **Administration** on the left menu (1)
* Select **System** tab (2)
* Enable JFFS partition (3)
* Ask to format the JFFS partition at the next boot (4)
* Enable SSH (5)
* Apply the changes ()
* ![JFFS&SSH](https://cloud.githubusercontent.com/assets/3483165/5581975/34677f0c-9030-11e4-922f-809fa46b5a9c.png)
* Reboot the router (to apply the jffs formating)
* ![boot](https://cloud.githubusercontent.com/assets/3483165/5582239/46bcdd56-9034-11e4-8e32-d40e7083f410.png)

* Install **Optware** (by installing, and then removing, Download Master if this application is not required). Optware is required to install **Rsync**.
* ![a](https://cloud.githubusercontent.com/assets/3483165/5582683/3c80bafe-903b-11e4-9187-3676d024ee02.png)
  * Depending if at this moment you have one or two usb devices connected to the router, the router will propose one or two usb devices to make the installation. Select the main one and proceed.
* ![a](https://cloud.githubusercontent.com/assets/3483165/5597595/05db30bc-9273-11e4-9c16-b3b2bee2f3bc.png)
* ![a](https://cloud.githubusercontent.com/assets/3483165/5582769/949a3eb2-903c-11e4-96cf-0886efa99e43.png)
  * Depending if you really need Download Master (a tool mainly used to download torrents ?), you can uninstall it right away. This procedure was just the easiest way to install Optware (by installing Download Master, you automatically install Optware).
 * ![a](https://cloud.githubusercontent.com/assets/3483165/5582803/0605f3de-903d-11e4-8af4-4da2337d4966.png)


***
> * You can also install Optware on a flash drive with this tutorial: 
> [https://github.com/RMerl/asuswrt-merlin/wiki/Initialize-OPTWARE]
> (https://github.com/RMerl/asuswrt-merlin/wiki/Initialize-OPTWARE)
> * Another tutorial: 
> [https://www.asuswrt.eu/how-to-install-optware/]
> (https://www.asuswrt.eu/how-to-install-optware/)

***

* Install a terminal emulator to access the router through a terminal emulator ([Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) or [Xshell](http://www.netsarang.com/download/down_form.html?code=522)) on your PC
> Optionally you can also install WinSCP on your PC to create folders and edit the files on the router (instead of using a line editor like **vi** or **nano** on the terminal emulator)

* "Connect" the terminal emulator to the router (usually 192.168.1.1, or router.asus.com)
* Using the terminal emulator (or WINscp), create 3 sub-folders to the jffs folder:

 * **config**
 * **scripts**
 * **dropbear**

* In the **scripts** folder, create the file **post-mount** . This script is called after a disk is mounted. You may have to check which disk just get mounted, That's why there is a "IF" section. Here is my own script. If you adapt it, please replace aaaa with the router **admin id**, XXXX with the remote router port number, and ZZZZ with the remote router ddns name). You can also remove completely the first "IF" section if there is no other partition on the main USB connected disk. 

> `#!/bin/sh`

> `#Start of the optional section (used if you have a second partition on the disk)`

> `if [  = "/tmp/mnt/RT-1080_rt" ]    # check if it is the post-mount of the partition reserved for the router. If you don't have it, this doesn't cause any problem. If you have it, and if a swap space is required, we will create that space here, in that partition [NOT USED FOR THE TUTORIAL]`

> `then`

> `#Turn on the swap space`

>  `swapon /tmp/mnt/RT-1080_rt/myswapfile`

> `#Set the _swappiness _ . This is the parameter that decice how aggressive is the swapping process. Could be set from 0 to 100. At 0 the router won't swap unless it hits the "no more available memory" state. By default: 60`

>  `echo 20 > /proc/sys/vm/swappiness`

> `fi`

> `#End of the optional section (used if you have a second partition on the disk). If you don't need it, just delete the lines starting at "Start of the optional section..."`

> `if [  = "/tmp/mnt/RT-1080" ]    # check if it is the post-mount of the main disk`

> `then`

> `#Turn on the swap space` for the case where the swap space have been created directly on the main disk

>  `swapon /tmp/mnt/RT-1080/myswapfile`

> `#Set the _swappiness _ . This is the parameter that decide how aggressive is the swapping process. Could be set from 0 to 100. At 0 the router won't swap unless it hits the "no more available memory" state. By default: 60`

>  `echo 20 > /proc/sys/vm/swappiness`

> `#first, bring back the "known_hosts"`

>  `rsync -avz --log-file=/mnt/RT-1080/Backup-logs/backup-known_hosts.log /mnt/RT-1080/bckp-1080/Router/ssh/ /root/.ssh`

> `#schedule the transfer of bckp-1080 to RT-8075. Here will be done daily at midnight`

>  `cru a backup1080to8075 "0 0 * * * rsync -avz -e 'ssh -p XXXX -i /jffs/dropbear/rsa_id' --log-file=/mnt/RT-1080/Backup-logs/backup1080to8075.log  /mnt/RT-1080/bckp-1080/ aaaaa@ZZZZZ.asuscomm.com:/mnt/RT-8075/bckp-1080"`

> `#schedule the transfer of bckp-8075 to RT-1080. Here will be done daily at 04h00`

>  `cru a backup8075to1080 "0 4 * * * rsync -avz -e 'ssh -p XXXX -i /jffs/dropbear/rsa_id' --log-file=/mnt/RT-1080/Backup-logs/backup8075to1080.log  aaaaa@ZZZZZ.asuscomm.com:/mnt/RT-8075/bckp-8075/ /mnt/RT-1080/bckp-8075"`

> `#make a backup of RT-1080 **jffs** a few times a day`

>   `cru a backupjffs1080 "0 8,12,18,23 * * * rsync -av --log-file=/mnt/RT-1080/Backup-logs/rsync-jffs-1080.log  /jffs /mnt/RT-1080/bckp-1080/Router"`

> `#make a backup of RT-8075 **jffs** a few times a day`

>   `cru a backupjffs8075 "30 8,12,18,23 * * * rsync -avz -e 'ssh -p XXXX-i /jffs/dropbear/rsa_id' --log-file=/mnt/RT-1080/Backup-logs/rsync-jffs-8075.log  aaaaa@YYYYYY.asuscomm.com:/jffs /mnt/RT-1080/bckp-8075/Router"`

> `#make a backup of RT-1080 "known_hosts" a few times a day`

>  `cru a backupknown_hosts "45 8,12,18,23 * * * rsync -avz --log-file=/mnt/RT-1080/Backup-logs/backup-known_hosts.log  /root/.ssh/ /mnt/RT-1080/bckp-1080/Router/ssh"`

> `fi`

> `exit $?`

* In the **scripts** folder, create the file **init-start** . Here is my ~own script. 
>
> `#!/bin/sh`
>
> `This script will create the 2 mounting points: one for the main partition, and one for the router partition. For the case you keeps the things simple and have just one disk and one poartition, just add a "#" in front of the second command, or just delete the line.`
>
> `mkdir -p /tmp/mnt/RT-1080`
>
> `mkdir -p /tmp/mnt/RT-1080_rt`


* When the scripts part is done, to make them executable, at the ssh terminal emulator enter:
> `chmod a+rx /jffs/scripts/*`

* In the **config** folder, create the file **fstab**. Here is my own content:

> `#Ntfs Backup disk (RT-1080); To get the UUID, you have to execute the command blkid on the ssh terminal. The UUID is use instead of the device path because the device path could change). For the understanding, UUID=01D01AD6E35757A0 could be replaced by /dev/sda1, as an example, but it is always more secure to work with the UUID. You can get the UUID by entering the "blkid" command at the terminal prompt.) `
>
>	`UUID=01D01AD6E35757A0 /tmp/mnt/RT-1080 tntfs rw,nodev,relatime,uid=0,gid=0,umask=00,nls=utf8,min_prealloc_size=64k,max_prealloc_size=3905928188,readahead=1M,user_xattr,case_sensitive,nocache,fail_safe,hidden=show,dotfile=show,errors=continue,mft_zone_multiplier=1 0 0`
>
> `#Ntfs Router partition (on the same disk) (optional)`
>
>	`UUID=f66b6687-d71a-d001-f04a-6087d71ad001 /tmp/mnt/RT-1080_rt tntfs rw,nodev,relatime,uid=0,gid=0,umask=00,nls=utf8, 0 0`


* Install Rsync. At the command line of the ssh terminal emulator, type:

> `ipkg upgrade`

> `ipkg install rsync`

***
**On the RT-8075 (remote router):** (uncompleted)

Enable ssh and jffs, and format jffs

* Select Administration on the left menu (1)
* Select System tab (2)
* Enable JFFS partition (3)
* Ask to format the JFFS partition at the next boot (4)
* Enable SSH (5)
* Allow SSH from WAN (6)
* Disallow SSH login (will use private&public keys instead of logging) (7)
* Apply the changes

* ![JFFS&SSH](https://cloud.githubusercontent.com/assets/3483165/5581975/34677f0c-9030-11e4-922f-809fa46b5a9c.png)

* Reboot the router (to apply the jffs re-formating)

* Connect a USB disk (ideally usb 3) on one of the usb port (ideally the usb 3 port if you use a usb3 disk...)
* Select "USB Applications" on the main menu of the GUI.
* Select "Media Services ans Servers" on the new page
* Select the tab labeled: "Network Place"
* Turn on "Enable Share" (4)
* For sake of simplicity at this moment, select "Allow guest login".(5) You can change it later and give specific rights to specific persons)
* Enter your specific "Work group" (6)
* Apply the changes (7)
   ![CreateShares1](https://cloud.githubusercontent.com/assets/3483165/5582437/aa41c190-9037-11e4-97c4-415e61fc668d.png)
   ![a](https://cloud.githubusercontent.com/assets/3483165/5597070/ff25c5ce-9269-11e4-890a-d30331138de6.png)

That done, on the same page, on the bottom part:
* Select the main disk (1)
* Create a share/folder "bckp-1080" for the local backup (2)
* Create a share/folder "bckp-8075" for storing the remote backup (2)
* Create a share/folder "bckp-logs for storing the logs of the backup processes (2)
* Optionally create a share/folder "Router" for the route/system needs (2)
* Apply the changes (3)
   ![a](https://cloud.githubusercontent.com/assets/3483165/5597134/222f2078-926b-11e4-9214-b6a7c0afac9b.png)

* Create an Asus ddns (xxx.asuscomm.com). This is to get a "permanent" URL pointing at the remote router.
* ![a](https://cloud.githubusercontent.com/assets/3483165/5597750/f6122ebc-9275-11e4-81b5-950b22bf9497.png)

* "Connect" the terminal emulator to the router (usually 192.168.1.1, or router.asus.com)
* Using the terminal emulator (or WINscp), create 3 sub-folders to the jffs folder:

 * **config**
 * **scripts**
 * **dropbear**

* In the **scripts** folder, create the file **post-mount** . This script is called after a disk is mounted. You may have to check which disk just get mounted, That's why there is a "IF" section. Here is my own script. If you adapt it, please replace aaaa with the router **admin id**, XXXX with the remote router port number, and ZZZZ with the remote router ddns name). You can also remove completely the first "IF" section if there is no other partition on the main USB connected disk. 

> `#!/bin/sh`

> `#Start of the optional section (used if you have a second partition on the disk)`

> `if [  = "/tmp/mnt/RT-8075_rt" ]    # check if it is the post-mount of the partition reserved for the router. If you don't have it, this doesn't cause any problem. If you have it, and if a swap space is required, we will create that space here, in that partition [NOT USED FOR THE TUTORIAL]`

> `then`

> `#Turn on the swap space`

>  `swapon /tmp/mnt/RT-8075_rt/myswapfile`

> `#Set the _swappiness _ . This is the parameter that decice how aggressive is the swapping process. Could be set from 0 to 100. At 0 the router won't swap unless it hits the "no more available memory" state. By default: 60`

>  `echo 20 > /proc/sys/vm/swappiness`

> `fi`

> `#End of the optional section (used if you have a second partition on the disk). If you don't need it, just delete the lines starting at "Start of the optional section..."`

> `if [  = "/tmp/mnt/RT-8075" ]    # check if it is the post-mount of the main disk`

> `then`

> `#Turn on the swap space` for the case where the swap space have been created directly on the main disk

>  `swapon /tmp/mnt/RT-8075/myswapfile`

> `#Set the _swappiness _ . This is the parameter that decide how aggressive is the swapping process. Could be set from 0 to 100. At 0 the router won't swap unless it hits the "no more available memory" state. By default: 60`

>  `echo 20 > /proc/sys/vm/swappiness`

> `fi`

> `exit $?`

* In the **scripts** folder, create the file **init-start** . Here is my ~own script. 
>
> `#!/bin/sh`
>
> `This script will create the 2 mounting points: one for the main partition, and one for the router partition. For the case you keeps the things simple and have just one disk and one poartition, just add a "#" in front of the second command, or just delete the line.`
>
> `mkdir -p /tmp/mnt/RT-8075`
>
> `mkdir -p /tmp/mnt/RT-8075_rt`


* When the scripts part is done, to make them executable, at the ssh terminal emulator enter:
> `chmod a+rx /jffs/scripts/*`

* In the **config** folder, create the file **fstab**. Here is my own content:

> `#Ntfs Backup disk (RT-8075); To get the UUID, you have to execute the command blkid on the ssh terminal. The UUID is use instead of the device path because the device path could change). For the understanding, UUID=01D01AD6E35757A0 could be replaced by /dev/sda1, as an example, but it is always more secure to work with the UUID. You can get the UUID by entering the "blkid" command at the terminal prompt.) `
>
>	`UUID=01D01AD6E35757A0 /tmp/mnt/RT-8075 tntfs rw,nodev,relatime,uid=0,gid=0,umask=00,nls=utf8,min_prealloc_size=64k,max_prealloc_size=3905928188,readahead=1M,user_xattr,case_sensitive,nocache,fail_safe,hidden=show,dotfile=show,errors=continue,mft_zone_multiplier=1 0 0`
>
> `#Ntfs Router partition (on the same disk) (optional)`
>
>	`UUID=f66b6687-d71a-d001-f04a-6087d71ad001 /tmp/mnt/RT-1080_rt tntfs rw,nodev,relatime,uid=0,gid=0,umask=00,nls=utf8, 0 0`


* Install Optware (please see instructions on the RT-1080 section)
* Install Rsync on the remote (please see instructions on the RT-1080 section)

***

When all that is done, then:
* Create and install the private and public rsa keys ([Using PuTTY Key Generator](http://the.earth.li/~sgtatham/putty/latest/x86/puttygen.exe))
![a](https://cloud.githubusercontent.com/assets/3483165/5589993/935707ee-90f8-11e4-9c8b-8ecd213694cc.png)

* Copy the public key to the **Authorized keys field** on the remote router(RT-8075) using the GUI (Administration/System)

* Save the private key and copy it to **/jffs/dropbear** under the name of: **rsa_id**
![a](https://cloud.githubusercontent.com/assets/3483165/5590022/25627790-90f9-11e4-9900-40be8bc9ee17.png)

* Test the keys to make sure it's possible to login on the remote (RT-8075) without a password. Enter this command line on the RT-1080 ssh terminal:

> *ssh -p XXXX -i /jffs/dropbear/rsa_id aaaaa@ZZZZZ.asuscomm.com

If everything is ok, you will be logged on the remote server. To logoff, simply type: exit and Return.
If it doesn't work, well...

On the RT-1080 terminal emulator, **please execute manually each rsync commands** listed on the post-mount script:

1. to make sure that there is no syntax error in the command
2. to confirm if you need a swap space, or not...

* Let's take one of the Rsync commands and use it to explain how it is built:

 * `rsync -avz -e 'ssh -p XXXX -i /jffs/dropbear/rsa_id' --log-file=/mnt/RT-1080/Backup-logs/backup1080to8075.log  /mnt/RT-1080/bckp-1080/ aaaaa@ZZZZZ.asuscomm.com:/mnt/RT-8075/bckp-1080`
 * -avz _[This will recursively transfer all files from the directory /mnt/RT-1080/bckp-1080/ on the local machine into the /mnt/RT-8075/bckp-1080 directory on the remote machine ZZZZZ.asuscomm.com: using the id aaaaa. The files are transferred in "archive" mode, which ensures that symbolic links, devices, attributes, permissions, ownerships, etc. are preserved in the transfer. Additionally, compression will be used to reduce the size of data portions of the transfer. The "v" is there to instruct Rsync to leave of trace of what it did.]_
 * -e 'ssh -p XXXX -i /jffs/dropbear/rsa_id'  _[XXXX is the remote ssh port and the remaining is the location of the private key]_
 * --log-file=/mnt/RT-1080/Backup-logs/backup1080to8075.log _[Where to log the things done]_
 * /mnt/RT-1080/bckp-1080/   _[Source]_
 * aaaaa@ZZZZZ.asuscomm.com:/mnt/RT-8075/bckp-1080  _[Destination. aaaaa is the remote admin id]_

If you get an error about the memory space (either from then sender or the receiver), then you will need to create a swap space on both ends. Because it is easier, and because there is no penalty to do it that way, we will create a swap file (instead of a swap partition).

***
> To create a swap file of 1GB on RT-1080 (the local and master router, directly on the main partition) (for 512MB, replace the 1024 with 512), enter the following commands on the terminal emulator connected to the router (wait for each command to complete; the first one could take a few minutes):
> * `dd if=/dev/zero of=/mnt/RT-1080/myswapfile bs=1M count=1024`
> * `chmod 600 /mnt/RT-1080/myswapfile`
> * `mkswap /mnt/RT-1080/myswapfile`
> * `swapon /mnt/RT-1080/myswapfile`
> * `free`
>
> Look at the output of the **free** command:
>
> `**                 total         used         free       shared      buffers`
>
> `**    Mem:        255776          85644         170132            0          768`
>
> `**    -/+ buffers:                84876         170900`

> `**     Swap:      1048572              0        1048572`

If the first number on the line starting with "swap" shows anything other than "0", then the swap is ok.
If everything is ok, then repeat the procedure on RT-8075 (the remote), and then test again the rsync commands.
***


Et voilà,

Gaston Huot