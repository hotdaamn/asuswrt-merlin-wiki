This is a partial list of features and changes that Asuswrt-merlin offers over the original Asus firmware.  Note that some features might not be available on every supported devices, due to limitations of the device.

**System:**
* Various bugfixes and optimizations
* Persistent JFFS partition
* Some components were updated to newerversions, for improved stability and security
* User scripts that run on specific events
* Cron jobs
* Ability to customize the config files used by the router services
* LED control - put your Dark Knight in Stealth Mode by turning off all LEDs
* Entware easy setup script (alternative to Optware - the two are mutually exclusive)
* SNMP support

**Disk sharing:**
* Optionally use shorter share names (folder name only)
* Disk spindown after user-configurable inactivity timeout
* NFS sharing (through webui)
* Entware easy setup script (alternative to Optware - the two are mutually exclusive)

**Networking:**
* Force acting as a Master Browser
* Act as a WINS server
* SSHD
* Allows tweaking TCP/UDP connection tracking timeouts
* CIFS client support (for mounting remote SMB share on the router)
* Layer7 iptables matching
* User-defined options for WAN DHCP queries (required by some ISPs)
* Improved NAT loopback (based on code from phuzi0n from the DD-WRT forums)
* Advanced OpenVPN client and server support (all models except RT-N16)
* Netfilter ipset module, for efficient blacklist implementation
* Wireless site survey
* Configurable IPv6 firewall
* Configurable min/max UPNP ports
* IPSec kernel support
* DNS-based Filtering, can be applied globally or per client
* Custom DDNS (through a user script)
* Advanced NAT loopback (as an alternative to the default one)

**Web interface:**
* Improved client list, with DHCP hostnames
* Optionally save traffic stats to disk (USB or JFFS partition)
* Enhanced traffic monitoring: added monthly, as well as per IP monitoring
* Name field on the DHCP reservation list and Wireless ACL list
* System info summary page
* Wireless client IP and hostname on the Wireless Log page
* Wifi icon reports the state of both radios
* Display the Ethernet port states
* The various MAC/IP selection pulldowns will also display hostnames when possible instead of just NetBIOS names
* Wireless site survey
* Advanced Wireless client list display, including automated refresh
