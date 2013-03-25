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
IPSET_PATH=/lib/modules/2.6.22.19/kernel/net/ipv4/netfilter
lsmod | grep "ipt_set" > /dev/null 2>&1 || \
for module in ip_set ip_set_nethash ip_set_iphash ipt_set
do
    insmod $IPSET_PATH/$module.ko
done

# Preparing folder to cache downloaded files
IPSET_LISTS_DIR=/jffs/ipset_lists
[ -d "$IPSET_LISTS_DIR" ] || mkdir -p $IPSET_LISTS_DIR

# Block traffic from Tor nodes
iptables -D INPUT -m set --set TorNodes src -j DROP
ipset --destroy TorNodes
ipset -N TorNodes iphash
[ -e $IPSET_LISTS_DIR/tor.lst ] || wget -q -O $IPSET_LISTS_DIR/tor.lst http://torstatus.blutmagie.de/ip_list_all.php/Tor_ip_list_ALL.csv
for IP in $(cat $IPSET_LISTS_DIR/tor.lst)
do
    ipset -A TorNodes $IP
done
iptables -A INPUT -m set --set TorNodes src -j DROP

# Block incoming traffic from some countries
iptables -D INPUT -m set --set BlockedCountries src -j DROP
ipset --destroy BlockedCountries
ipset -N BlockedCountries nethash
# pk and cn is for Pakistan and China. See other country code at http://www.ipdeny.com/ipblocks/
for country in pk cn
do
    [ -e $IPSET_LISTS_DIR/$country.lst ] || wget -q -O $IPSET_LISTS_DIR/$country.lst http://www.ipdeny.com/ipblocks/data/countries/$country.zone
    for IP in $(cat $IPSET_LISTS_DIR/$country.lst)
    do
        ipset -A BlockedCountries $IP
    done
done
iptables -A INPUT -m set --set BlockedCountries src -j DROP
```
and make it executable:
```
chmod +x /jffs/scripts/firewall-start
```
You may run `/jffs/scripts/firewall-start` from command line or reboot router to apply new blocking rules immediately. 