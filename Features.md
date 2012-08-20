This is a partial list of features and changes that Asuswrt-merlin offers over the original Asus firmware.  Some features are for specific devices.  Other times, features available on a device by Asus have been made available to other devices.

* Various bugfixes such as crashing issues related to VPN, etc...
* 64KB NVRAM (RT-N66U; RT-AC66U is already 64KB)
* WakeOnLan web interface (with user-entered preset targets)
* JFFS persistent partition
* User scripts executed at init, services startup, WAN up, firewall up and shutdown.
* SSHD (through dropbear)
* HTTPS support
* OUI (MAC address) lookup if you click on a MAC on the Client list (ported from DD-WRT)
* Optionally turn the WPS button into a radio on/off switch
* Saving your traffic history to disk (USB or JFFS)
* Displaying monthly traffic history
* Cron jobs
* Monitor your router's temperature (under Administration -> Performance Tuning)
* Display active/tracked network connections
* Allows tweaking TCP/UDP connection tracking timeouts
* layer7 and cifs kernel modules added
* Optional user-settings for the WAN DHCP client (required by some ISPs)
* Description field added to DHCP reservation entries
* Dual WAN support (both failover and load balancing) (EXPERIMENTAL feature developed by Asus)
* Improved NAT loopback
* Disk spindown after user-configurable inactivity timeout
* System info summary page
* Wireless client IP and hostname on the Wireless Log page
