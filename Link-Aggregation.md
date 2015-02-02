Starting with firmware version Asuswrt-Merlin 3.0.0.4.374.33 bonding.ko Driver is included in the firmware.

**There are several modes which should be possible.**

1. balance-rr or 0
2. active-backup or 1
3. balance-xor or 2
4. broadcast or 3
5. 802.3ad or 4
6. balance-tlb or 5
7. balance-alb or 6 

**The only mode that has been tested is 802.3ad**

NIC bonding in mode 802.3ad achieves 2 goals.

1. Increased Throughput
2. Failover Protection

But requires either a Switch/PC/NAS which also supports Link Aggregation.

***

**METHOD 1 - LINKAGG (Custom Script)**

To Simplify setup and management a small shell script has been created.

Which can be downloaded

[LinkAgg](http://www.mediafire.com/download/8y3ye3332durchp/LinkAgg)

Usage is as follows.

        ----  Link Aggregation Version 1.5 Help  ----
	Dynamically enable Link Aggregation using 802.3ad
	802.3ad requires a Switch/PC/NAS which...
	also supports 802.3ad to function correctly
	Usage: /path/to/LinkAgg <port> <port>
	Example: /path/to/LinkAgg 3 4
	Only 2 ports are currently supported
        --- Special Flags ---
	Help: -h or --help
	Status: -s or --status
	Delete: -d or --delete
	Version: -v or --version

**To Make the Configuration persistent across reboots.**

1. Enable JFFS partition
2. Edit /jffs/scripts/services-start to call the script at each boot

`/Path/to/LinkAgg 3 4`

**Additional Notes.**

Any additional configuration can be done manually in the services-start script

The Script above will use the sysfs driver because it does not require any additional packages.

Optionally users can choose to install entware - ifenslave package.

*** entware is for MIPS based routers only ***

More Info Here http://forums.smallnetbuilder.com/showthread.php?t=12735

***

**METHOD 2 - For AC68U/R/P Routers**

**Step 1 - NVRAM Edits** | Note: You will need to repeat this step if you clear the nvram ie. Beta to Final versions, resetting to Factory default

Apply the following changes to the router's nvram:

    nvram set vlan4ports="3 5t"
    nvram set vlan5ports="4 5t"
    nvram set vlan4hwname=et0
    nvram set vlan5hwname=et0
    nvram commit

**Step 2 - Create/Edit services-start script**

Include the following code in services-start script located in /jffs/scripts/ (you will need to create this file from scratch, if you haven't done so already, with the right permissions)

    #!/bin/sh

    # Logger Services
    logger -t "($(basename ))" $$ SERVICES-START being started....

    logger -t "($(basename ))" $$ Bonding ports 3 and 4 commencing....
    # Pre-Bonding
    robocfg vlan 1 ports "1 2 5*"

    # Bonding
    sleep 2s
    modprobe bonding
    # Setting mode to 802.3ad
    echo 802.3ad > /sys/class/net/bond0/bonding/mode
    # Setting LACP rate to fast
    echo fast > /sys/class/net/bond0/bonding/lacp_rate
    # Setting MII monitoring interval to 50
    echo 50 > /sys/class/net/bond0/bonding/miimon
    # Setting xmit hash policy to layer3+4
    echo 1 > /sys/class/net/bond0/bonding/xmit_hash_policy
    ip link set bond0 up
    echo +vlan4 > /sys/class/net/bond0/bonding/slaves
    echo +vlan5 > /sys/class/net/bond0/bonding/slaves
    brctl addif br0 bond0

    # Post-Bonding
    sleep 2s
    logger -t "($(basename $0))" $$ Bonding Status....
    cat /proc/net/bonding/bond0 | sed 's/^/+++ /' | logger

**Step 3 - Create/Edit firewall-start script**

Include the following code in firewall-start script located in /jffs/scripts/ (you will need to create this file from scratch, if you haven't done so already, with the right permissions)

    #!/bin/sh

    # Bonding IPtables rules
    iptables -I INPUT -i vlan4 -j ACCEPT
    iptables -I INPUT -i vlan5 -j ACCEPT
    iptables -I INPUT -i bond0 -j ACCEPT

   # Firewall/IPtables Performance Tweak for Bond0 to be placed right after the above bonding rules and 
   # before your custom rules - if any.
   iptables -D INPUT `iptables --line-numbers -nL INPUT | grep ESTABLISHED | tail -n1 | awk '{print $1}'`
   iptables -I INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

**Step 4 - Set the switch's LAG hashing mode**

This is the LAG config profile you created for the 2 switch ports you have assigned to the router's port 3 and 4.

Set the hashing mode to "Source/Destination MAC, VLAN, EtherType, source MODID/port" or the equivalent mode in your switch.

**Step 5 - Reboot**

**Additional Notes.**

This method may be adapted to work with other Asus routers. Hint: Internal switch port mappings.

More info Here http://forums.smallnetbuilder.com/showthread.php?t=20441
