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

Starting with Asuswrt-Merlin 3.0.0.4.374.32, a user-configurable IPv6 firewall is available (and is enabled by default).  Be aware however that previous versions, as well as the original firmware from Asus provide NO firewalling on the IPv6 interfaces, leaving your whole LAN open for access from the Internet over IPv6 addresses.


### Conclusion

That pretty much covers it.  Tunnels are a great way for you to familiarize yourself with IPv6, regardless of what your ISP provides you.
