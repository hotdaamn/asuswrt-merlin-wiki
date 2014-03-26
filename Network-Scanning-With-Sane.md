This page describes how to enable Network Scanning using a USB scanner or multi-function printer in Merlin using S.A.N.E. (Scan Access Now Easy).

Requirements:
* A router running Merlin duh...
* USB Stick with optware configured
* A Multi-Function USB Scanner or Multi-Function Printer
* Router is accessible via ssh

Router Setup Instructions:
* ssh to router and type **opkg install sane-backends sane-frontends sane-libs
 dbus xinetd**
* create file **/opt/etc/xinetd.d/saned** and put the following content:
`service saned
{
socket_type = stream
server = /opt/sbin/saned
protocol = tcp
user = admin
group = root
wait = no
disable = no
}
`


 