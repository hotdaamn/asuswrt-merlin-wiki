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
iptables-save | grep TorNodes > /dev/null 2>&1 || iptables -I INPUT -m set --set TorNodes src -j DROP

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
iptables-save | grep BlockedCountries > /dev/null 2>&1 || iptables -I INPUT -m set --set BlockedCountries src -j DROP
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
iptables -I FORWARD -m set --set BluetackLevel1 src,dst -j DROP
```
and run it:
```
sh /jffs/scripts/peerguardian.sh
```
Please don't close SSH-session until it finishes. Script will blocks [over 8 000 000 IP's addresses](http://www.iblocklist.com/list.php?list=bt_level1) which anti-p2p activity has been seen from.