# Using ipset

Since 3.0.0.4_270.26 [ipset](http://en.wikipedia.org/wiki/Netfilter#ipset) feature has been implemented. This is an Netfilter extension which able:
* to store multiple IP addresses or port numbers and match against the collection by iptables at one swoop;
* to dynamically update iptables rules against IP addresses or ports without performance penalty;
* to express complex IP address and ports based rulesets with one single iptables rule and benefit from the speed of IP sets.

This is an example of using [ipset utility](http://manpages.ubuntu.com/manpages/lucid/man8/ipset.8.html) with two different set types: iphash and nethash. The example shows how to block incoming connection form [Tor](https://www.torproject.org/) nodes (iphash set type — number of ip addresses) and how to block incoming connection from whole countries (nethash set type - number of ip subnets). 

Please, enable and format [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) through WEB UI, place this content to `/jffs/scripts/firewall-start`

```
#!/bin/sh

# Loading ipset modules
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_nethash ip_set_iphash ipt_set
do
    insmod $module
done

# Preparing folder to cache downloaded files
IPSET_LISTS_DIR=/jffs/ipset_lists
[ -d "$IPSET_LISTS_DIR" ] || mkdir -p $IPSET_LISTS_DIR

# Different routers got different iptables syntax
case $(uname -m) in
  armv7l)
    MATCH_SET='--match-set'
    ;;
  mips)
    MATCH_SET='--set'
    ;;
esac

# Block traffic from Tor nodes
if [ "$(ipset --swap TorNodes TorNodes 2>&1 | grep 'Unknown set')" != "" ]
then
    ipset -N TorNodes iphash
    [ -e $IPSET_LISTS_DIR/tor.lst ] || wget -q -O $IPSET_LISTS_DIR/tor.lst http://torstatus.blutmagie.de/ip_list_all.php/Tor_ip_list_ALL.csv
    for IP in $(cat $IPSET_LISTS_DIR/tor.lst)
    do
        ipset -A TorNodes $IP
    done
fi
[ -z "$(iptables-save | grep TorNodes)" ] && iptables -I INPUT -m set $MATCH_SET TorNodes src -j DROP

# Block incoming traffic from some countries. cn and pk is for China and Pakistan. See other countries code at http://www.ipdeny.com/ipblocks/
if [ "$(ipset --swap BlockedCountries BlockedCountries 2>&1 | grep 'Unknown set')" != "" ]
then
    ipset -N BlockedCountries nethash
    for country in pk cn
    do
        [ -e $IPSET_LISTS_DIR/$country.lst ] || wget -q -O $IPSET_LISTS_DIR/$country.lst http://www.ipdeny.com/ipblocks/data/countries/$country.zone
        for IP in $(cat $IPSET_LISTS_DIR/$country.lst)
        do
            ipset -A BlockedCountries $IP
        done
    done
fi
[ -z "$(iptables-save | grep BlockedCountries)" ] && iptables -I INPUT -m set $MATCH_SET BlockedCountries src -j DROP

# Block Microsoft telemetry spying servers
if [ "$(ipset --swap MicrosoftSpyServers MicrosoftSpyServers 2>&1 | grep 'Unknown set')" != "" ]
then
    ipset -N MicrosoftSpyServers iphash
    for IP in 23.99.10.11 63.85.36.35 63.85.36.50 64.4.6.100 64.4.54.22 64.4.54.32 64.4.54.254 \
              65.52.100.7 65.52.100.9 65.52.100.11 65.52.100.91 65.52.100.92 65.52.100.93 65.52.100.94 \
              65.55.29.238 65.55.39.10 65.55.44.108 65.55.163.222 65.55.252.43 65.55.252.63 65.55.252.71 \
              65.55.252.92 65.55.252.93 66.119.144.157 93.184.215.200 104.76.146.123 111.221.29.177 \
              131.107.113.238 131.253.40.37 134.170.52.151 134.170.58.190 134.170.115.60 134.170.115.62 \
              134.170.188.248 157.55.129.21 157.55.133.204 157.56.91.77 168.62.187.13 191.234.72.183 \
              191.234.72.186 191.234.72.188 191.234.72.190 204.79.197.200 207.46.223.94 207.68.166.254
    do
        ipset -A MicrosoftSpyServers $IP
    done
fi
[ -z "$(iptables-save | grep MicrosoftSpyServers)" ] && iptables -I FORWARD -m set $MATCH_SET MicrosoftSpyServers dst -j DROP
```
and make it executable:
```
chmod +x /jffs/scripts/firewall-start
```
You may run `/jffs/scripts/firewall-start` from command line or reboot router to apply new blocking rules immediately. 


***
## Peer Guardian
Another example is a [PeerGuardian](http://en.wikipedia.org/wiki/PeerGuardian) functionality right on router.

Please do not add this script to `/jffs/scripts/firewall-start` because it executes too long (~25 min on RT-N66U). Place following content to `/jffs/scripts/peerguardian.sh`
```
#!/bin/sh

# Loading ipset modules
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_iptreemap ipt_set
do
    insmod $module
done

# Different routers got different iptables syntax
case $(uname -m) in
  armv7l)
    MATCH_SET='--match-set'
    ;;
  mips)
    MATCH_SET='--set'
    ;;
esac

# PeerGuardian rules
if [ "$(ipset --swap BluetackLevel1 BluetackLevel1 2>&1 | grep 'Unknown set')" != "" ]
then
    ipset --create BluetackLevel1 iptreemap
    [ -e /tmp/bluetack_lev1.lst ] || wget -q -O - "http://list.iblocklist.com/?list=bt_level1&fileformat=p2p&archiveformat=gz" | \
        gunzip | cut -d: -f2 | grep -E "^[-0-9.]+$" > /tmp/bluetack_lev1.lst
    for IP in $(cat /tmp/bluetack_lev1.lst)
    do
        ipset -A BluetackLevel1 $IP
    done
fi
iptables -I FORWARD -m set $MATCH_SET BluetackLevel1 src,dst -j DROP
```
and run it:
```
sh /jffs/scripts/peerguardian.sh
```
Please don't close SSH-session until it finishes. Script will blocks [over 8 000 000 IP's addresses](http://www.iblocklist.com/list.php?list=bt_level1) which anti-p2p activity has been seen from.

***
## Peer Guardian V2
Below is a speed optimized version of the peerguardian.sh script above. It does the same thing, but takes less than 30 seconds to run (the shortest run took 20 seconds on my RT-N66U). It might now be possible to run it from `/jffs/scripts/firewall-start`.

The script utilizes two sets: primary "BluetackLevel1" and temporary "BluetackLevel2". The IPs are bulk loaded into the temporary one and then swapped into the primary. Because of this approach it can also be run periodically on a running router to refresh the active set.
```
#!/bin/sh

# PeerGuardian rules

# Loading ipset modules
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_iptreemap ipt_set; do
    insmod $module
done

# Different routers got different iptables syntax
case $(uname -m) in
  armv7l)
    MATCH_SET='--match-set'
    ;;
  mips)
    MATCH_SET='--set'
    ;;
esac

# Create the BluetackLevel1 (primary) if does not exists
if [ "$(ipset --swap BluetackLevel1 BluetackLevel1 2>&1 | grep 'Unknown set')" != "" ]; then
  ipset --create BluetackLevel1 iptreemap && \
  iptables -I FORWARD -m set $MATCH_SET BluetackLevel1 src,dst -j DROP
fi
# Destroy this transient set just in case
ipset --destroy BluetackLevel2 > /dev/null 2>&1
    
# Load the latest rule(s)

(echo -e "-N BluetackLevel2 iptreemap\n" && \
 nice wget -q -O - "http://list.iblocklist.com/?list=bt_level1&fileformat=p2p&archiveformat=gz" | \
    nice gunzip | nice cut -d: -f2 | nice grep -E "^[-0-9.]+$" | \
    nice sed 's/^/-A BluetackLevel2 /' && \
 echo -e "\nCOMMIT\n" \
) | \
nice ipset --restore && \
nice ipset --swap BluetackLevel2 BluetackLevel1 && \
nice ipset --destroy BluetackLevel2

exit $?
```

##Peerguardian V3
If you want to have different blocklist, grouped by one, then this is a variant, where you can add multiple blocklists in one script...

```
#!/bin/sh

logger "PeerGuardian rules"

logger "Loading ipset modules"
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_iptreemap ipt_set; do
    insmod $module
done

case $(uname -m) in
  armv7l)
    MATCH_SET='--match-set'
    ;;
  mips)
    MATCH_SET='--set'
    ;;
esac

logger "Create the BluetackLevel1 (primary) if does not exists"
if [ "$(ipset --swap BluetackLevel1 BluetackLevel1 2>&1 | grep 'Unknown set')" != "" ]; then
  ipset --create BluetackLevel1 iptreemap && \
  iptables -I FORWARD -m set $MATCH_SET BluetackLevel1 src,dst -j DROP
fi
logger "Destroy this transient set just in case"
ipset --destroy BluetackLevel2 > /dev/null 2>&1

logger "Load the latest rule(s)"

(
	 
	(
		(
		 nice wget -q -O - "http://list.iblocklist.com/?list=dgxtneitpuvgqqcpfulq&fileformat=p2p&archiveformat=gz" | \
	         nice gunzip | nice cut -d: -f2 | nice grep -E "^[-0-9.]+$"  \
		) && \
		(
	 	 nice wget -q -O - "http://list.iblocklist.com/?list=llvtlsjyoyiczbkjsxpf&fileformat=p2p&archiveformat=gz" | \
        	 nice gunzip | nice cut -d: -f2 | nice grep -E "^[-0-9.]+$"  \
		) && \
		(
		 nice wget -q -O - "http://list.iblocklist.com/?list=ydxerpxkpcfqjaybcssw&fileformat=p2p&archiveformat=gz" | \
		 nice gunzip | nice cut -d: -f2 | nice grep -E "^[-0-9.]+$"  \
		)
	) | \
	(  
      		nice sed '/^$/d' | \
	        nice sed 's/^/-A BluetackLevel2 /' | \
		nice sed '1s/^/-N BluetackLevel2 iptreemap\n/' && \
		echo -e "\nCOMMIT\n" \
	)
#) > output
) | \
nice ipset --restore && \
nice ipset --swap BluetackLevel2 BluetackLevel1 && \
nice ipset --destroy BluetackLevel2

logger "exiting Peerguarding rules"
exit $?
```
The output will be in router logs:
```
May 29 09:03:21 admin: PeerGuardian rules
May 29 09:03:21 admin: Loading ipset modules
May 29 09:03:21 admin: Create the BluetackLevel1 (primary) if does not exists
May 29 09:03:22 admin: Destroy this transient set just in case
May 29 09:03:22 admin: Load the latest rule(s)
May 29 09:04:04 admin: exiting Peerguarding rules
```