## Introduction ##

*This guide is adapted from a [how-to](http://forums.smallnetbuilder.com/showthread.php?t=9449) posted by [ryzhov_al](http://forums.smallnetbuilder.com/member.php?u=13498) on the [SmallNetBuilder forums](http://forums.smallnetbuilder.com). The guide provided here on the wiki has been modified to include translation enhancements and to better describe the steps involved, however the commands provided are identical.*

This guide will help you setup advertisement filtering (similar to [Adblock Plus](http://adblockplus.org)) through your router. It is suggested to use in conjunction with a mobile device (such as an iOS or Android smartphone), as filtering a PC will cause performance loss due to hardware limitations on the router. This guide uses [privoxy](http://www.privoxy.org) (a proxy server) to capture and filter traffic from mobile devices.

## Prerequisites ##

To setup advertisement filtering on your router, you will need the following:

* An Asuswrt-Merlin compatible router with Asuswrt-Merlin installed
* A USB disk connected to the router
* A working Entware environment. Instructions to setup Entware can be found in [this guide](https://github.com/RMerl/asuswrt-merlin/wiki/Entware).

## Installation ##

First, install the necessary packages through Entware:

    opkg install bash wget sed privoxy

Next, download and install the already prepared privoxy configuration file:

    cd /opt/etc/privoxy/
    rm ./config
    wget http://files.ryzhov-al.ru/Routers/adblock-plus/config

Next, download and install the script that will convert Adblock Plus rules for use with our proxy:

    wget http://files.ryzhov-al.ru/Routers/adblock-plus/privoxy-blocklist_0.2.sh

Mark the script as executable so we can run it:

    chmod +x ./privoxy-blocklist_0.2.sh

Edit the script to include the Adblock Plus subscriptions that you would like to use. A list of subscriptions can be found on [this page](http://adblockplus.org/en/subscriptions). You may also consult the Adblock Plus installation in your preferred browser for the subscriptions it uses. Find the *URLS=* line in the script and append the URLs of the subscriptions to it.

Next, download the subscription's filters and create the privoxy blacklist by running the script:

    ./privoxy-blocklist_0.2.sh

Determine the device you would like to apply advertisement filtering to. You will need to apply a static IP for this device - you can do so by accessing your router's web interface (the default address is 192.168.1.1) in a web browser, then clicking *LAN* in the navigation sidebar (under *Advanced Settings*), then clicking the *DHCP Server* tab. Enable the *Enable Manual Assignment* radio option, and select or input the MAC address of the device in the *Manually Assigned IP around the DHCP list* dropdown box. If the device you wish to filter does not appear in the dropdown box, make sure that it is connected to the router via a wired or wireless connection and refresh the page. Assign the MAC address a static IP address within the router's IP address range (for example, 192.168.1.25). Be sure to click the *Apply* button to apply your changes.

If you have already set up manually assigned IP addresses via DHCP you can find them in one of the following files:

    cat /etc/hosts.dnsmasq
    cat /var/lib/misc/dnsmasq.leases

Next, add a rule to iptables to intercept traffic from the device and pass it through privoxy for advertisement filtering:

    echo \#!/bin/sh > /jffs/scripts/firewall-start
    echo iptables -t nat -A PREROUTING --source [the static IP address you provided] -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 3128 >> /jffs/scripts/firewall-start
    chmod +x /jffs/scripts/firewall-start

*(Ensure you replace [the static IP address you provided] with the static IP address you assigned in the previous step.)*

Reboot the router, and check the device to ensure that advertisement filtering is working as expected!

## Changing or Updating the Subscriptions ##

If you'd like to change the subscriptions (for example, to add a new subscription list) or update the existing subscriptions, you'll need to remove the old ones first:

    ./privoxy-blocklist_0.2.sh -r

If you're changing the subscription lists, edit the script as described in the instructions above to add/remove the subscription URLs as necessary. If you just wish to update the lists, no script editing is necessary. Then run the script to download the updated subscription filters and create a new privoxy blacklist:

    ./privoxy-blocklist_0.2.sh

## Conclusion ##

At this point, you should have a working proxy server that filters advertisements from webpages loaded on a provided device. For discussion and assistance if this guide is not working for you, please refer to the [thread this guide came from](http://forums.smallnetbuilder.com/showthread.php?t=9449) on the SmallNetBuilder forums.