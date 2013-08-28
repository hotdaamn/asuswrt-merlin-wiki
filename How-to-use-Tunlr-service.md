### Introduction ###

This guide will help you use [Tunlr service](http://tunlr.net/) to watch Netflix, MTV, CBS, Hulu & more outside the U.S.

### Prerequisites ###

* An Asuswrt-Merlin compatible router with Asuswrt-Merlin installed,
* [JFFS enabled](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS).

### Installation ###

Type

    cd /
    wget -O - http://files.ryzhov-al.ru/Routers/dnmasq%20and%20Tunlr/tunlr_update.tgz | tar -xvz

and reboot router.

### Details ###

There is a shell script `/jffs/scripts/wan-start` to update Tunlr's DNS servers after internet connection has been established:

    #!/bin/sh
    PATH='/usr/local/sbin:/usr/sbin:/usr/bin:/sbin:/bin'
    # Tunlr DNS updater for dnsmasq. 2013, Alexander Ryzhov
    # Adapted For asuswrt-merlin firmware.
    logger -t $(basename $0) "started [$@]"
    DNSMASQ_CONF=/jffs/configs/dnsmasq.conf.add
    DOMAINS=$(cat /jffs/configs/domains.txt)
    IPS=$(wget -q -O - \
        "http://tunlr.net/tunapi.php?action=getdns&version=1&format=json" \
        | sed "s/\"dns.\"://g" | sed "s/[{}]//g" | sed "s/,/\ /g" |sed "s/\"//g")
        if [ -z "$IPS" ] || [ -n "$(echo $IPS | sed 's/[0-9\.\ ]//g')" ] ; then
            echo "Tunlr DNS addresses not retrieved, exiting."
        exit
    fi
    echo -n > $DNSMASQ_CONF
    for domain in $DOMAINS
    do
        for dns in $IPS
        do
            echo "server=/${domain}/${dns}" >> $DNSMASQ_CONF
        done
    done
    service restart_dnsmasq

It takes Tunlr's DNS addresses and makes dnsmasq configuration file where some domains will be "served" via Tunlr service. Feel free to add your own domains to `/jffs/configs/domains.txt` file.