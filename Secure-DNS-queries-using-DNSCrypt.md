Here is another tutorial about enabling dnscrypt on asuswrt routers.

[Install Entware](https://github.com/RMerl/asuswrt-merlin/wiki/Entware#the-easy-way), then install necessary packages:

```
opkg install dnscrypt-proxy fake-hwclock
```

Tell router to use new resolver:
```
echo "no-resolv" > /jffs/configs/dnsmasq.conf.add
echo "server=127.0.0.1#65053" >> /jffs/configs/dnsmasq.conf.add
```

You can block using other DNS-servers on clients (optional):
```
echo "#!/bin/sh" >> /jffs/scripts/firewall-start
echo "iptables -A OUTPUT -p tcp --dport 53 -j DROP" >> /jffs/scripts/firewall-start
chmod +x /jffs/scripts/firewall-start
```
Reboot router and make sure it works by visiting [this page](http://www.opendns.com/support/article/64)

More info here http://www.snbforums.com/threads/dnscrypt-from-opendns.11645/