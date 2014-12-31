> ### Before going further, please note that you may prefer (if it works) use a similar builtin feature:
> ![Builtin](https://cloud.githubusercontent.com/assets/3483165/5582361/bf776b74-9036-11e4-9844-85912b2993a0.png)
***

###  That said, in the following tutorial, we will learn how to setup the transfer of a backup to a remote site using a ssh channel between 2 ASUS routers and Rsync as transporter.

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
* not creating the swap space (which could be not required if the backup is not "that big").

Let's start the procedure: (draft)
***
On RT-1080 (local):
* Connect a USB disk (ideally usb 3) on one of the usb port (ideally the usb 3 port if you use a usb3 disk...)
* Create a share/folder "Backup" for the local backup
* Create a share/folder "Backup-8075" for storing the remote backup
* Create a share/folder "Router" for the route/system needs **(not absolutely needed)**
 * ![CreateShares1](https://cloud.githubusercontent.com/assets/3483165/5582437/aa41c190-9037-11e4-97c4-415e61fc668d.png)
 * ![shares](https://cloud.githubusercontent.com/assets/3483165/5582530/12526d38-9039-11e4-9113-d1f8a4f862fc.png)
* Enable ssh and jffs and format jffs
 1. Select Administration on the left menu
 2. Select System tab
 3. Enable JFFS partition
 4. Ask to format the JFFS partition at the next boot
 5. Enable SSH
* ![ssh](https://cloud.githubusercontent.com/assets/3483165/5581944/b00b9806-902f-11e4-90c9-c783b87807d3.png)
* Apply the changes
* Reboot the router (to apply the jffs formating)
* ![boot](https://cloud.githubusercontent.com/assets/3483165/5582239/46bcdd56-9034-11e4-8e32-d40e7083f410.png)
* Install Optware (by installing, and then removing, Download Master if not required)
* ![a](https://cloud.githubusercontent.com/assets/3483165/5582683/3c80bafe-903b-11e4-9187-3676d024ee02.png)
   * The router will propose one or two usb devices to make the installation. Select one and proceed.
 * ![a](https://cloud.githubusercontent.com/assets/3483165/5582702/78c8d78a-903b-11e4-87db-e9b8f426a3fb.png)
 * ![a](https://cloud.githubusercontent.com/assets/3483165/5582769/949a3eb2-903c-11e4-96cf-0886efa99e43.png)
   * Depending if you really need Dowmnload master (a tool mainly used to download torrents ?), you can install it right away. This installation was just was the easiest way to install Optware.
 * ![a](https://cloud.githubusercontent.com/assets/3483165/5582803/0605f3de-903d-11e4-8af4-4da2337d4966.png)
 * Here is an alternate way for the Optware installation: [https://github.com/RMerl/asuswrt-merlin/wiki/Initialize-OPTWARE](https://github.com/RMerl/asuswrt-merlin/wiki/Initialize-OPTWARE)
* Install a terminal to access the router through ssh (Putty or Xshell) on your PC
> * Optionally you can also install WinSCP on your PC to edit the files on the router (instead of using VI or Nano using the terminal)
* Connect to the router (usually 192.168.1.1, or https://router.asus.com)
* Create two sub-folders to the jffs folder:
 * config
  * fstab
   * ...
 * scripts
  * post-mount

> `#!/bin/sh`

> `if [  = "/tmp/mnt/Routeur" ]    # check si c'est le post-mount de la partition réservée au routeur`

> `then`

> `#Active le swap space`

> `# touch /tmp/activeswap.txt `

>  `swapon /tmp/mnt/Routeur/myswapfile`

> `#Fixe le swappiness qui est le paramètre qui détermine le degré d'agressivité du processus de swap. Peut varier de 0 à 100. À 0 le routeur ne va swapper que si manque total de mémoire. Par défaut: 60`

>  `echo 20 > /proc/sys/vm/swappiness`

> `fi`

> `#`

> `if [  = "/tmp/mnt/My_Book" ]    # check si c'est le post-mount de My_Book USB drive`

> `then`

> `#récupère les "known_hosts"`

>  `rsync -avz --log-file=/mnt/My_Book/Backup-logs/backup-known_hosts.log /mnt/My_Book/Backup/1080-AsusRouter/ssh/ `

> `/root/.ssh`

> `#cédule le backup externe du 1080 vers le 8075`

>  `cru a backup1080to8075 "0 0 * * * rsync -avz -e 'ssh -p 1220 -i /jffs/dropbear/dropbear_rsa_host_key' --log-file=/mnt/My_Book/Backup-logs/backup1080to8075.log  /mnt/My_Book/Backup/ admin@boudor.asuscomm.com:/mnt/My_Book/Backup-1080"`

> `#cédule le backup externe du 8075 vers le 1080`

>  `cru a backup8075to1080 "0 4 * * * rsync -avz -e 'ssh -p 1220 -i /jffs/dropbear/dropbear_rsa_host_key' --log-file=/mnt/My_Book/Backup-logs/backup8075to1080.log  admin@boudor.asuscomm.com:/mnt/My_Book/Backup/ /mnt/My_Book/Backup-8075"`

> `#fait un backup de la partition permanente jffs du 1080 4 fois par jour`

>   `cru a backupjffs1080 "0 8,12,18,23 * * * rsync -av --log-file=/mnt/My_Book/Backup-logs/rsync-jffs-1080.log  /jffs /mnt/My_Book/Backup/1080-AsusRouter"`

> `#fait un backup de la partition permanente jffs du 8075 4 fois par jour`

>   `cru a backupjffs8075 "30 8,12,18,23 * * * rsync -avz -e 'ssh -p 1220 -i /jffs/dropbear/dropbear_rsa_host_key' --log-file=/mnt/My_Book/Backup-logs/rsync-jffs-8075.log  admin@boudor.asuscomm.com:/jffs /mnt/My_Book/Backup/8075-AsusRouter"`

> `#fait un backup des "known_hosts" 4 fois par jou`r

>  `cru a backupknown_hosts "45 8,12,18,23 * * * rsync -avz --log-file=/mnt/My_Book/Backup-logs/backup-known_hosts.log  /root/.ssh/ /mnt/My_Book/Backup/1080-AsusRouter/ssh"`

> `fi`

> `exit $?`

* Install Rsync. At the command line type:
> ipkg install rsync

***
On the RT-8075 (remote)
* Connect a USB disk (ideally usb 3) on one of the usb port (ideally the usb 3 port if you use a usb3 disk...)
* Create a share/folder "Backup" for the local backup
* Create a share/folder "Backup-8075" for storing the remote backup
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
* Install a terminal to access the router through ssh (Putty or Xshell)
* Connect to the router
* Create two sub-folders to the jffs folder:
 * config
 * scripts
* Install Rsync


Then:
* Create and install the private and public rsa keys for both routers (using Easy-RSA)
* Test the keys to make sure it's possible to login without a password

Please execute first your rsync commands manually:

1. to make sure that there is no syntax error in the command
1. to confirm if you need a swap space or not

If you get an error about a problem with the memory space (either from then sender or the receiver), then you will need to create a swap space on both ends.
Because it is easier, and there is no penalty to do it that way, we will create a swap file, instead of a swap partition.

To create a swap file on RT-1080 (local):
* ...
* ...
* ...

Repeat the procedure on RT-8075 (remote)
chmod a+rx /jffs/scripts/services-start
/jffs/scripts/services-start