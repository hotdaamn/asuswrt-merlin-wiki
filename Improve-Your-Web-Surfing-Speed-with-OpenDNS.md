Here is another tutorial by [ryzhov_al](http://forums.smallnetbuilder.com/member.php?u=13498) about enabling OpenDNS on asuswrt routers

1 - Install [Entware](https://github.com/RMerl/asuswrt-merlin/wiki/Entware)
2 - Download the latest packages:
```
opkg update && opkg upgrade 
```
3 - Install dnscrypt:
```
opkg install dnscrypt-proxy dnscrypt-proxy-hostip
```
4 - Tell router to use it:
```
echo "no-resolv" > /jffs/configs/dnsmasq.conf.add
echo "server=127.0.0.1#65053" >> /jffs/configs/dnsmasq.conf.add
```
5 - Paste this content to /jffs/scripts/wan-start:
```
#!/bin/sh

# Wait up to 15 seconds to make sure /opt partition is mounted
i=0
while [ $i -le 15 ]
do
    if [ -d /opt/tmp ]
    then
        break
    fi
    sleep 1
    i=`expr $i + 1`
done

# Now resolve DNS name for NTP server
rm -f /jffs/configs/hosts.add
ntp_name=$(nvram get ntp_server0)
for ip in $(/opt/sbin/dnscrypt-proxy-hostip $ntp_name)
do
    echo $ip $ntp_name >> /jffs/configs/hosts.add
done

# and restart NTP client to eliminate 4-5 mins delay
killall ntp && sleep 1
service restart_ntpc
```
6 - Make the script executable:
```
chmod +x /jffs/scripts/wan-start
```
7 - Reboot router and make sure it works by visiting [this page](http://www.opendns.com/support/article/64)

More info here http://forums.smallnetbuilder.com/showthread.php?t=11645