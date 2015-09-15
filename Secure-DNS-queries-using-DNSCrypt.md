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

...
Save the time for fake-hwclock initially.  Run this:
...
/opt/bin/fake-hwclock
...

(optional) You can block using other DNS-servers on clients:
```
echo "#!/bin/sh" >> /jffs/scripts/firewall-start
echo "iptables -A OUTPUT -p tcp --dport 53 -j DROP" >> /jffs/scripts/firewall-start
chmod +x /jffs/scripts/firewall-start
```

Reboot router to take effect:
```
reboot
```

More info and discussion [here](http://www.snbforums.com/threads/dnscrypt-from-opendns.11645/).