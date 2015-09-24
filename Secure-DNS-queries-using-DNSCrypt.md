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

(optional) You can redirect using other DNS-servers on clients:
add to firewall-start or nat-start
```
iptables -t nat -A PREROUTING -i br0 -p udp --dport 53 -j DNAT --to $(nvram get lan_ipaddr)
iptables -t nat -A PREROUTING -i br0 -p tcp --dport 53 -j DNAT --to $(nvram get lan_ipaddr)
```

Reboot router to take effect:
```
reboot
```

More info and discussion [here](http://www.snbforums.com/threads/dnscrypt-from-opendns.11645/).