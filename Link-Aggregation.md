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

**To Simplify setup and management a small shell script has been created.**

Which can be downloaded, future firmware versions will include a copy of this script.

[LinkAgg](http://www.mediafire.com/download/kc298caz28y8kda/LinkAgg)

Usage is as follows.

        ----  Link Aggregation Version 1.3 Help  ----
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
