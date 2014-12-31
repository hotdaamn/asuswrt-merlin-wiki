***
> ### Before going further, please note that you may prefer (if it works) use a similar built-in feature:
> ![Builtin](https://cloud.githubusercontent.com/assets/3483165/5582361/bf776b74-9036-11e4-9844-85912b2993a0.png)
***

###  That said, in the following tutorial, we will learn how to setup the transfer of a backup to a remote site using a ssh channel between 2 ASUS routers and Rsync as transporter.

> Context: 

> * on a site (1080), someone makes on the local router usb disk a daily backup of all his PCs important files.
> * on another site (8075), someone makes on the local router usb disk a daily backup of all his PCs important files.
> * each of them wants to place a copy of its local backup to a remote site, meaning that each of them wants to daily transfer its backup to the other router usb disk.
> * when possible, all commands come from 1080.

> If you want something different, please adapt the rsync commands and the script to your needs.

 [Rsync](http://en.wikipedia.org/wiki/Rsync) is a "classic" linux program used to synchronize/copy efficiently data between two locations. The 2 locations could be "local", even on the same server/router, but our goal being to secure a backup, we will do it to a remote location. We will also secure the transfer itself by using [SSH ](http://en.wikipedia.org/wiki/Secure_Shell). Secure Shell (SSH) is a cryptographic network protocol for secure data communication, remote command-line login, remote command execution, etc. To make it even more secure, we will use a private/public key schema (instead of id/password).

In my specific case, the routers are named:
* RT-1080
* RT-8075

Both need Rsync installed. RT-1080 is the "master", i.e. the one responsible for all actions going on between the 2 routers and RT-8075 is the "slave", just reacting to RT-1080 requests. Each router has a 4TB USB3 disk (on the USB3 port, indeed). Each disk has 2 partitions: 
* the "data partition" (~4TB)
* the "router partition" (~5GB)

> **Please note that you don't absolutely need the "router partition".**

To **Rsync**, you will possibly need a [swap space](http://en.wikipedia.org/w/index.php?title=Paging) because the router memory could be too limited for the process, depending on the size of the backup. If required, the swap space could be set on a file located on the router partition. Optware/rsync is also installed on that partition. Simply said, [Optware](http://en.wikipedia.org/wiki/Optware) is a mechanism that simplifies the installation of some programs on the router.

The data partition is used to receive:
* the local backup: bckp-1080
* the remote backup: bckp-8075

In my specific case, [Windows File History](http://www.pcmag.com/article2/0,2817,2418904,00.asp) is the tool used to backup two local PC on RT-1080, and same thing on the remote side, on RT-8075.  The goal is therefore to send bckp-1080 to RT-18075, and bckp-8075 to RT-1080. The backups will be scheduled to run daily at night using CRON. [CRON](http://en.wikipedia.org/wiki/Cron) is the standard Linux tool used to schedule processes on a periodic way.
The CRU command will be used to setup the CRON job.

I also use 
* **JFFS**, 
* **fstab**, one of the [Custom config files](https://github.com/RMerl/asuswrt-merlin/wiki/Custom-config-files) and 
* the ["User scripts"](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) of RMerlin firmware version. Two scripts will be setup:
 * **init-start**
 * **post-mount**

[JFFS](http://en.wikipedia.org/wiki/JFFS) is a [writable section of the flash memory](https://github.com/RMerl/asuswrt-merlin/wiki/Jffs) used to store some user's data/program/scripts. Intrinsically, the router is a device with no permanent memory, memory being erased on each shutdown, with the exception of the JFFS space. Please note that the JFFS space has to be also backup because **it could be** overwritten by a new firmware version.

[Fstab](http://en.wikipedia.org/wiki/Fstab) is a linux configuration file used to mount the disks. This file will be stored in the folder **Config** located under **JFFS**. The scripts are event-triggered and will be used to initiate some actions on the router reboot, and on the post-mount of the disk/partitions. The **User's scripts** are located under the folder "**scripts**" also under **JFFS**. Therefore, when backing up the JFFS space, at the same time I backup **fstab** and the **scripts**.

We will go first with the most simple approach:
* using the backup disk as "system disk", meaning that we will use the usb backup disk to install all the required elements.
* not creating the swap space (because it could be not required if your backup is not "that big").

***
Let's start the procedure: (draft)
***
On RT-1080 (local):
* Connect a USB disk (ideally usb 3) on one of the usb port (ideally the usb 3 port if you use a usb3 disk...)
* Create a share/folder "Backup" for the local backup
* Create a share/folder "Bckp-8075" for storing the remote backup

> * Create a share/folder "Router" for the route/system needs **(not absolutely needed)**

   ![CreateShares1](https://cloud.githubusercontent.com/assets/3483165/5582437/aa41c190-9037-11e4-97c4-415e61fc668d.png)
   ![shares](https://cloud.githubusercontent.com/assets/3483165/5582530/12526d38-9039-11e4-9113-d1f8a4f862fc.png)

* Enable ssh and jffs and format jffs
 1. Select **Administration** on the left menu
 2. Select **System** tab
 3. Enable JFFS partition
 4. Ask to format the JFFS partition at the next boot
 5. Enable SSH

* ![ssh](https://cloud.githubusercontent.com/assets/3483165/5581944/b00b9806-902f-11e4-90c9-c783b87807d3.png)
* Apply the changes
* Reboot the router (to apply the jffs formating)
* ![boot](https://cloud.githubusercontent.com/assets/3483165/5582239/46bcdd56-9034-11e4-8e32-d40e7083f410.png)

* Install **Optware** (by installing, and then removing, Download Master if not required). Optwre is require to install **Rsync**.
* ![a](https://cloud.githubusercontent.com/assets/3483165/5582683/3c80bafe-903b-11e4-9187-3676d024ee02.png)
  * The router will propose one or two usb devices to make the installation. Select one and proceed.
* ![a](https://cloud.githubusercontent.com/assets/3483165/5582702/78c8d78a-903b-11e4-87db-e9b8f426a3fb.png)
* ![a](https://cloud.githubusercontent.com/assets/3483165/5582769/949a3eb2-903c-11e4-96cf-0886efa99e43.png)
  * Depending if you really need Dowmnload Master (a tool mainly used to download torrents ?), you can uninstall it right away. This procedure was just the easiest way to install Optware (by installing Download Master, you automatically install Optware).
 * ![a](https://cloud.githubusercontent.com/assets/3483165/5582803/0605f3de-903d-11e4-8af4-4da2337d4966.png)


***
> * Here is an alternate way for the Optware installation: 
> [https://github.com/RMerl/asuswrt-merlin/wiki/Initialize-OPTWARE]
> (https://github.com/RMerl/asuswrt-merlin/wiki/Initialize-OPTWARE)

***


* Install a terminal to access the router through ssh (Putty or Xshell) on your PC
> Optionally you can also install WinSCP on your PC to create folders and edit the files on the router (instead of using a line editor like Vi or Nano on the ssh terminal)

* Connect the terminal to the router (usually 192.168.1.1, or router.asus.com)
* Using the terminal (or WINscp), create two sub-folders to the jffs folder:

 * **config**
 * **scripts**

* In the **scripts** folder, create the file **post-mount** . Here is my own script. If you use it, please replace aaaa with the router **admin id**, XXXX with the remote router port number, and ZZZZ with the remote router ddns name):

> `#!/bin/sh`

> `if [  = "/tmp/mnt/RT-1080" ]    # check if it is the post-mount of the partition reserved for the router`

> `then`

> `#Turn on the swap space`

>  `swapon /tmp/mnt/RT-1080/myswapfile`

> `#Set the _swappiness _ . This is the parameter that decice how aggressive is the swapping process. Could be set from 0 to 100. At 0 the router won't swap unless it hits the "no more available memory" state. By default: 60`

>  `echo 20 > /proc/sys/vm/swappiness`

> `fi`

> `if [  = "/tmp/mnt/My_Book" ]    # check if it is the post mount of the main disk`

> `then`

> `#first, bring back the "known_hosts"`

>  `rsync -avz --log-file=/mnt/My_Book/Backup-logs/backup-known_hosts.log /mnt/My_Book/Backup/RT-1080/ssh/ /root/.ssh`

> `#schedule the transfer of bckp-1080 to RT-8075`

>  `cru a backup1080to8075 "0 0 * * * rsync -avz -e 'ssh -p XXXX -i /jffs/dropbear/dropbear_rsa_host_key' --log-file=/mnt/My_Book/Backup-logs/backup1080to8075.log  /mnt/My_Book/Backup/ aaaaa@ZZZZZ.asuscomm.com:/mnt/My_Book/bckp-1080"`

> `#schedule the transfer of bckp-8075 to RT-1080`

>  `cru a backup8075to1080 "0 4 * * * rsync -avz -e 'ssh -p XXXX -i /jffs/dropbear/dropbear_rsa_host_key' --log-file=/mnt/My_Book/Backup-logs/backup8075to1080.log  aaaaa@ZZZZZ.asuscomm.com:/mnt/My_Book/Backup/ /mnt/My_Book/Backup-8075"`

> `#make a backup of RT-1080 **jffs** a few times a day`

>   `cru a backupjffs1080 "0 8,12,18,23 * * * rsync -av --log-file=/mnt/My_Book/Backup-logs/rsync-jffs-1080.log  /jffs /mnt/My_Book/Backup/RT-1080"`

> `#make a backup of RT-8075 **jffs** a few times a day`

>   `cru a backupjffs8075 "30 8,12,18,23 * * * rsync -avz -e 'ssh -p XXXX-i /jffs/dropbear/dropbear_rsa_host_key' --log-file=/mnt/My_Book/Backup-logs/rsync-jffs-8075.log  aaaaa@YYYYYY.asuscomm.com:/jffs /mnt/My_Book/Backup/bckp-8075"`

> `#make a backup of "known_hosts" a few times a day`

>  `cru a backupknown_hosts "45 8,12,18,23 * * * rsync -avz --log-file=/mnt/My_Book/Backup-logs/backup-known_hosts.log  /root/.ssh/ /mnt/My_Book/Backup/RT-1080/ssh"`

> `fi`

> `exit $?`

* In the **scripts** folder, create the file **init-start** . Here is my own script. 
>
> `#!/bin/sh`
>
> `This script will create the 2 mounting points, if you are going to have a specific disk for the router needs. Otherwise, add a "#" in front of the second command, or just delete the line.`
>
> `mkdir -p /tmp/mnt/My_Book`
>
> `mkdir -p /tmp/mnt/RT-1080`


* When the scripts are done, to make them executable, on the ssh terminal enter:
> `chmod a+rx /jffs/scripts/*`

* In the **config** folder, create the file **fstab**. Here is my own content:

> `#Ntfs Backup disk (My_Book); To get the UUID, you have to execute the command blkid on the ssh terminal. The UUID is use instead of the device path because the device path could change). For the understanding, UUID=01D01AD6E35757A0 could be replaced by /dev/sda1, as an example) `
>
>	`UUID=01D01AD6E35757A0 /tmp/mnt/My_Book tntfs rw,nodev,relatime,uid=0,gid=0,umask=00,nls=utf8,min_prealloc_size=64k,max_prealloc_size=3905928188,readahead=1M,user_xattr,case_sensitive,nocache,fail_safe,hidden=show,dotfile=show,errors=continue,mft_zone_multiplier=1 0 0`
>
> `#Ntfs Router partition (on the same disk) (not absolutely required)`
>
>	`UUID=f66b6687-d71a-d001-f04a-6087d71ad001 /tmp/mnt/RT-1080 tntfs rw,nodev,relatime,uid=0,gid=0,umask=00,nls=utf8, 0 0`


* Install Rsync. At the command line type:

> ipkg install rsync

***
On the RT-8075 (remote)
* Connect a USB disk (ideally usb 3) on one of the usb port (ideally the usb 3 port if you use a usb3 disk...)
* Create a share/folder "Backup" for the local backup
* Create a share/folder "bckp-1080" for storing the remote backup
* Create a share/folder "Router" for the route/system needs
* Enable ssh and jffs and format jffs
 1. Select Administration on the left menu
 2. Select System tab
 3. Enable JFFS partition
 4. Ask to format the JFFS partition at the next boot
 5. Enable SSH
 6. Allow SSH from WAN
 7. Disallow SSH login (will use private&public keys instead of logging)
* ![JFFS&SSH](https://cloud.githubusercontent.com/assets/3483165/5581975/34677f0c-9030-11e4-922f-809fa46b5a9c.png)
* Apply the changes
* Reboot the router (to apply the jffs re-formating)
* Create an Asus ddns (xxx.asuscomm.com)
* ![a](https://cloud.githubusercontent.com/assets/3483165/5582292/5fa50a4a-9035-11e4-810b-802bd24aa0c7.png)
* Install Optware (by installing, and then removing, Download Master)
* Connect the ssh terminal to the router.
* Create two sub-folders to the jffs folder:
 * config
 * scripts

* Install Rsync


***

Then:
* Create and install the private and public rsa keys for both routers (using Easy-RSA)
* Test the keys to make sure it's possible to login on the remote (RT-8075) without a password
> *ssh 

At the RT-1080 ssh terminal, please execute each rsync commands manually, the ones on the post-mount script:

1. to make sure that there is no syntax error in the command
2. to confirm if you need a swap space or not

If you get an error about the memory space (either from then sender or the receiver), then you will need to create a swap space on both ends. Because it is easier, and there is no penalty to do it that way, we will create a swap file, instead of a swap partition.

To create a swap file of 1GB on RT-1080 (local, directly on the main partition) (for 512MB, replace the 1024 with 512), enter the following commands at the ssh terminal connected to the router (wait for each command to complete; the first one could take a few minutes):
* dd if=/dev/zero of=/mnt/RT-1080/myswapfile bs=1M count=1024
* chmod 600 /mnt/RT-1080/myswapfile
* mkswap /mnt/RT-1080/myswapfile
* swapon /mnt/RT-1080/myswapfile
* free
**              total         used         free       shared      buffers
** Mem:        255776        85644       170132            0          768
** -/+ buffers:              84876       170900
** Swap:      1048572            0      1048572

Repeat the procedure on RT-8075 (the remote), and then test again the rsync commands.



Gaston Huot