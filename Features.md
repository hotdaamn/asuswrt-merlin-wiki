This is a partial list of features and changes that Asuswrt-merlin offers over the original Asus firmware.  Note that some features might not be available on every supported devices, due to limitations of the device.

**System:**
* Various bugfixes (like the crash on VPN/NAT Loopback access of LAN devices)
* Persistent JFFS partition
* User scripts that run on specific events
* Cron jobs
* Customized config files for router services
* LED control - put your Dark Knight in Stealth Mode by turning off all LEDs

**Disk sharing:**
* Optionally use shorter share names (folder name only)
* Disk spindown after user-configurable inactivity timeout

**Networking:**
* WakeOnLan web interface (with user-entered preset targets)
* SSHD
* Allows tweaking TCP/UDP connection tracking timeouts
* CIFS client support (for mounting remote SMB share on the router)
* Layer7 iptables matching
* User-defined options for WAN DHCP queries (required by some ISPs)
* Improved NAT loopback (based on code from phuzi0n from the DD-WRT forums)
* Dual WAN support (both failover and load balancing supported) (EXPERIMENTAL) (RT-N66U, RT-AC66U)
* OpenVPN client and server, based on code originally written by Keith Moyer for Tomato and reused with his permission. (RT-N66U, RT-AC66U)

**Web interface:**
* Clicking on the MAC address of an unidentified client will do a lookup in the OUI database (ported from DD-WRT).
* Optionally save traffic stats to disk (USB or JFFS partition)
* Display monthly traffic reports
* Display active/tracked network connections
* Name field on the DHCP reservation list and Wireless ACL list
* System info summary page
* Wireless client IP, hostname, rate and rssi on the Wireless Log page
* Wifi icon reports the state of both radios
