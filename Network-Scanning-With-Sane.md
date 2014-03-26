This page describes how to enable Network Scanning using a USB scanner or multi-function printer in Merlin using S.A.N.E. (Scan Access Now Easy).

### Requirements:
* A router running Merlin duh...
* USB Stick with optware configured
* A Multi-Function USB Scanner or Multi-Function Printer
* Router is accessible via ssh

### Router Setup Instructions:
* ssh to router and type **opkg install sane-backends sane-frontends sane-libs
 dbus xinetd**
* create file **/opt/etc/xinetd.d/saned** and put the following content:  
**service saned**  
**{**  
**socket_type = stream**  
**server = /opt/sbin/saned**  
**protocol = tcp**  
**user = admin**  
**group = root**  
**wait = no**  
**disable = no**  
**}**  
* edit file **/opt/etc/dbus-1/system.conf** and replace the following line:  
**`<user>root</user>`**  
with  
**`<user>admin</user>`**  
* Reboot router and make sure that dbus and xinetd are running (this step might not be required if you know how to start the services manually from /opt/etc/init.d)

### Special Hardware Instructions
### HP:
* **opkg --force-overwrite install hplip**

### Test functionality:
* type: **scanimage -L**, your device should be displayed
* type: **scanimage --test**, your device should start testing and display status

### Om The Client Side (Linux):
* install the sane package. This changes by distribution
* edit file **/etc/sane.d/net.conf** and add the ip or hostname of the router
* test the same way with the **scanimage** utility


Suggestions are always welcomed.
 