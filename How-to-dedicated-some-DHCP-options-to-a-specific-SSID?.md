# Identify SSID Interface
As the following configuration is applied to a specific interface, you need to identify which interface / virtual interface your SSID is using, to do so you can use the following command:
`nvram show | grep WiFiTest`

Which should result something similar to:
 wl0.2_ssid=WiFiTest
 size: 48528 bytes (17008 left)
 wl_ssid=WiFiTest

This example give us wl0.2 as the interface that we need to tweak.

# Scripts
## DNSMasq Script
Create the following file: /jffs/scripts/dnsmasq.postconf
Give it the following permission: chmod +xxx /jffs/scripts/dnsmasq.postconf
Its content should be:
 #!/bin/sh
 CONFIG=$1
 source /usr/sbin/helper.sh
 logger "dnsmasq-dhcp: Configure wl0.2 to have special DHCP"
 ifconfig wl0.2 172.30.20.2 netmask 255.255.255.0
 iptables -D INPUT -i wl0.2 -j ACCEPT
 iptables -I INPUT -i wl0.2 -j ACCEPT
 ebtables -t broute -D BROUTING -i wl0.2 -p ipv4 -j DROP
 ebtables -t broute -I BROUTING -i wl0.2 -p ipv4 -j DROP
 pc_append "
 log-dhcp
 interface=wl0.2
 dhcp-option=wl0.2,3,172.30.20.1
 dhcp-option=wl0.2,6,8.8.8.8,8.8.4.4
 " /tmp/etc/dnsmasq.conf

Feel free to customize the DHCP option and range.

## Service-start Script
You need to create this file to ensure the DHCP service is restarted after the WiFi interface have been mounted.
Create the following file: `/jffs/scripts/services-start`
Give it the following permission: `chmod +xxx /jffs/scripts/services-start`
Its content should be:
 #!/bin/sh
 service restart_dnsmasq`

## Debug
You can restart the dnsmasq server by doing service:
`service restart_dnsmasq`

It does avoid to reboot the router.

Also ensure you DHCP configuration have the following configuration
`log-dhcp`

It will produce more advance logs that could help you.  