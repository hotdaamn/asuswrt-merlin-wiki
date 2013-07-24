This is a partial list of features and changes that Asuswrt-merlin offers over the original Asus firmware.  Note that some features might not be available on every supported devices, due to limitations of the device.

**System:**
* Various bugfixes (like the crash on VPN/NAT Loopback access of LAN devices)
* Persistent JFFS partition
* User scripts that run on specific events
* Cron jobs
* Customized config files for router services
* LED control - put your Dark Knight in Stealth Mode by turning off all LEDs
* Entware easy setup script (alternative to Optware - the two are mutually exclusive)

**Disk sharing:**
* Optionally use shorter share names (folder name only)
* Disk spindown after user-configurable inactivity timeout
* Entware easy setup script (alternative to Optware - the two are mutually exclusive)

**Networking:**
* Act as a Master Browser
* Act as a WINS server
* SSHD
* Allows tweaking TCP/UDP connection tracking timeouts
* CIFS client support (for mounting remote SMB share on the router)
* Layer7 iptables matching
* User-defined options for WAN DHCP queries (required by some ISPs)
* Improved NAT loopback (based on code from phuzi0n from the DD-WRT forums)
* Dual WAN support (both failover and load balancing supported) (EXPERIMENTAL) (RT-N66U, RT-AC66U)
* OpenVPN client and server, based on code originally written by Keith Moyer for Tomato and reused with his permission. (RT-N66U, RT-AC66U)
* Netfilter ipset module, for efficient blacklist implementation
* Wireless site survey

**Web interface:**
* Optionally save traffic stats to disk (USB or JFFS partition)
* Display monthly traffic reports
* Display historical traffic sorted by local IP
* Name field on the DHCP reservation list and Wireless ACL list
* System info summary page
* Wireless client IP, hostname, rate and rssi on the Wireless Log page
* Wifi icon reports the state of both radios
* Display the Ethernet port states
* The various MAC/IP selection pulldowns will also display hostnames when possible instead of just NetBIOS names
