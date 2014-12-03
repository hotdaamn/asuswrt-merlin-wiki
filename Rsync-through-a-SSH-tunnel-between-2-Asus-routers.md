* Enable ssh on both routers
* Create an Asus ddns on both routers (xxx.asuscomm.com)
* Install a terminal to access the router through ssh (Putty or Xshell)
* Connect to the router
* Enable JFFS
* Install Optware (by installing, and then removing, Download Master)
* Install Rsync
* Create and install private and public rsa keys for both routers (using Easy-RSA)
* Test the keys to make sure it's possible to login without a password
* Create a script to run at boot time to schedule the backup
* Create the Rsync script
* Schedule backup
* 
chmod a+rx /jffs/scripts/services-start
/jffs/scripts/services-start
