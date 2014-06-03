Quite a few tricks can be done through the implementation of a few iptable rules either in the firewall-start scripts or the nat-start scripts.

These tricks will require first that you enable the JFFS partition, and then that you put your rules in one of the user scripts (either firewall-start or nat-start, depending on the case).

### Allow port forwarding to a service (like RDesktop) only from a specific IP

Let's say you want to create a port forward that will only accept connections from a specific IP (for example, you want to only allow RDesktop connection coming from IP 10.10.10.10).  First, make sure you do NOT forward that port on the Virtual server page.  Then, use a rule like this inside the nat-start script:

```
iptables -t nat -I VSERVER 3 -p tcp -m tcp -s 10.10.10.10 --dport 3389 -j DNAT --to 192.168.1.100
```
This will forward connections coming from 10.10.10.10 to your PC running RDesktop, on IP 192.168.1.100.

The same method can be used to allow forwarding SSH, FTP, etc...  Just adjust the --dport to match the service port you want to forward to.