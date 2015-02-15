Quite a few tricks can be done through the implementation of a few iptable rules either in the firewall-start scripts or the nat-start scripts.

These tricks will require first that you enable the JFFS partition, and then that you put your rules in one of the user scripts (either firewall-start or nat-start, depending on the case).

### Allow port forwarding to a service (like RDesktop) only from a specific IP

Let's say you want to create a port forward that will only accept connections from a specific IP (for example, you want to only allow RDesktop connection coming from IP 10.10.10.10).  First, make sure you do NOT forward that port on the Virtual server page.  Then, use a rule like this inside the nat-start script:

```
iptables -t nat -I VSERVER 3 -p tcp -m tcp -s 10.10.10.10 --dport 3389 -j DNAT --to 192.168.1.100
```
This will forward connections coming from 10.10.10.10 to your PC running RDesktop, on IP 192.168.1.100.

The same method can be used to allow forwarding SSH, FTP, etc...  Just adjust the --dport to match the service port you want to forward to.

### Configure a port forward with brute force detection

The following nat-start script will create a port forward that will only forward connections if there is no more than a set number of attempts within a period if time.  This example will forward SSH to an internal server sitting on 192.168.1.100, limiting connection attempts at 5 per period of 60 seconds:

```
#!/bin/sh

logger "firewall" "Applying nat-start rules"
iptables -N SSHVSBFP -t nat
iptables -A SSHVSBFP -t nat -m recent --set --name SSHVS --rsource
iptables -A SSHVSBFP -t nat -m recent --update --seconds 60 --hitcount 5 --name SSHVS --rsource -j RETURN
iptables -A SSHVSBFP -t nat -p tcp --dport 22 -m state --state NEW -j DNAT --to-destination 192.168.1.100:22
iptables -I VSERVER -t nat -i eth0 -p tcp --dport 22 -m state --state NEW -j SSHVSBFP
```

Make sure to remove any existing port forward rule - this script will take care of creating the forward rule.

