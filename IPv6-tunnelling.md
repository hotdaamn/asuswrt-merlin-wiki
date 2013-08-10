### Introduction

Asuswrt comes with built-in support for numerous tunnelling methods that allows you to get IPv6 support through your IPv4-only ISP.   You can use a 6in4 tunnel provided by Hurricane Electrics.


### Hurricane Electrics (AKA TunnelBroker) service

Signing up for a tunnel with them is free, and simple.  They will provide you with a routed /64 block (meaning you get your own static, fully routed, block of 2^64 IPs).  You can also request a larger block if needed (for example if you want to do some additional subnetting).

Setting up the router is fairly simple.  On the IPv6 section simply select the 6in4 option, and fill up the informations provided by HE:

* IPv4 remote Endpoint = Server IPv4 Address on TunnelBroker
* IPv6 client IP = Client IPv6 Address (without the /64 at the end - that is entered below)
* IPv6 Prefix Lenght will be 64
* IPv6 Prefix = Routed /64: (without the /64 at the end)
* Prefix Lenght will be 64
* IPv6 DNS Server 1 is optional (if your ISP's DNS support IPv6 resolution), otherwise enter the Anycasted IPv6 Caching Nameserver

Also make sure to enable Router Advertisement.  This is what will automatically assign IPv6 addresses to your devices on your LAN, and provide them with routing information (a bit similar to what DHCP does for IPv4, but more advanced).

The next step: you need to somehow tell Hurricane Electrics what your current IPv4 is.  There are two methods.

1) You can use the TunnelBroker option on the DDNS tab.  Just like a regular Dynamic DNS service, this will take care of letting HE know about any IP change.

2) If you want to use DDNS for a regular DDNS service, you can replace the functionality using a custom script.


### Configuring a tunnel endpoint update script

For the second method, you will need to enable the [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) partition.  Once that's done, create a [wan-start](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) user script that will take care of updating HE with your current WAN IP.  Here is an example script:

```
#!/bin/sh
#v1.40-rm3 Mar 24, 2012
#***************************
#Settings start here
#***************************

#account info to auto update endpoint
USERID="this_is_your_hexadecimal_userid_from_tunnelbroker"
PASSWD="your_tunnelbroker_password"
TUNNELID="12345_from_tunnelbroker"

#####Optional/Advanced Settings######

#HE's endpoint verificiation server ip to add to whitelist
HE_VERIFY_SERVER_IP="66.220.2.74"

#logging settings (set to /dev/null for no logging)
STARTUP_SCRIPT_LOG_FILE="/tmp/ipv6.log"

#***************************
#Settings end here
#***************************


########################
#Tunnel endpoint update
########################
echo "" >> $STARTUP_SCRIPT_LOG_FILE
echo "HE IPv6 Script started" >> $STARTUP_SCRIPT_LOG_FILE
#get a hash of the plaintext password
MD5PASSWD=`echo -n $PASSWD | md5sum | sed -e 's/  -//g'`
echo `date` >> $STARTUP_SCRIPT_LOG_FILE
echo "configuring tunnel" >> $STARTUP_SCRIPT_LOG_FILE

#update HE endpoint
#need to alllow wan ping or HE will not validate new endpoint
#iptables -I INPUT 2 -s $HE_VERIFY_SERVER_IP -p icmp -j ACCEPT
etime=`date +%s`

wget -q "http://ipv4.tunnelbroker.net/ipv4_end.php?ip=AUTO&pass=$MD5PASSWD&apikey=$USERID&tid=$TUNNELID" -O /tmp/wget.tmp.$etime

cat /tmp/wget.tmp.$etime >> $STARTUP_SCRIPT_LOG_FILE
echo "" >> $STARTUP_SCRIPT_LOG_FILE
rm /tmp/wget.tmp.$etime
```

Edit the parameters at the top to enter your UserID (it's an hexadecimal value, found on the Main Page of your account info in Tunnel Broker - right after you log in), password and tunnelID (found on the page where you see all the details of your tunnel).

You can manually execute the script, then check the content of /tmp/ipv6.log to ensure that the script is running correctly.

After that, you can test your tunnel by accessing, for example, http://www.whatismyipv6.com.  If you see an IPv6 (meaning a long hexadecimal string), then congratulations - you're set!


### Security

By default, Asuswrt (and Asuswrt-Merlin) comes with NO firewalling on the IPv6 side of things.  That means that any computer or device obtaining an IPv6 will be directly reachable from the Internet.  A future release of Asuswrt-Merlin will add an IPv6 firewall.  In the mean time, you will have to provide your own firewall rules.

Here is an example script for manually configuring a very basic IPv6 firewall:

```
#############
# Firewalling
#############
ip6tables -A INPUT -j DROP

ip6tables -I FORWARD 2 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Allowed inbound rules here, such as this one:
#ip6tables -I FORWARD 2 -p tcp -m state --state NEW -i v6in4 -d 2001:123:44:555:6666:7777:8888:9999 --dport 3389 -j ACCEPT
ip6tables -A FORWARD -i v6in4 -o br0 -p all -j DROP
ip6tables -A FORWARD -i br0 -o any -p all -j ACCEPT
ip6tables -A FORWARD -i br0 -o v6in4 -p all -j ACCEPT
ip6tables -A FORWARD -i any -o br0 -p all -j ACCEPT
ip6tables -A FORWARD -j DROP
```

The first rule in the FORWARD chain (which is commented out) shows an example on how to open a port to a specific IP on your network (remember to enter the computer's IP in that rule, NOT your WAN IP - you are obtaining an entire IP block here, you are NOT using NAT like you probably do with your unique IPv4!).

Save that script as a [firewall-start](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) script.


### Conclusion

That pretty much covers it.  Tunnels are a great way for you to familiarize yourself with IPv6, regardless of what your ISP provides you.
