### Introduction
If you set the DDNS (dynamic DNS) service to "Custom", then you will be able to fully control the update process through a ddns-start user script.  That script could launch a custom DDNS update client, or run a simple "wget" on a provider's update URL.  The ddns-start script will be passed the WAN IP as an argument.

Note that the script will also be responsible for notifying the firmware on the success or failure of the process.  To do this you must simply run the following command:

```
/sbin/ddns_custom_updated 0 or 1
```

0 = failure, 1 = successful update

If you cannot determine the success or failure, then report it as a success to ensure that the firmware won't continuously try to force an update.

Finally, like all custom scripts, the option to support custom scripts and config files must be enabled under Administration -> System.


Below you will find a collection of scripts to handle additional DDNS services:

### afraid.org

Here is a working example, for afraid.org's free DDNS (you must update the URL to use your private API key from afraid.org)

```
#!/bin/sh

wget -q http://freedns.afraid.org/dynamic/update.php?your-private-key-goes-here -O - >/dev/null

if [ $? -eq 0 ]; then
    /sbin/ddns_custom_updated 1
else
    /sbin/ddns_custom_updated 0
fi
```

### DNSexit.com

Free DNS server that also offers DDNS services.

```
#!/bin/sh
/usr/bin/wget -qO - http://update.dnsexit.com/RemoteUpdate.sv?login=<username>&password=<password>&host=<domain>&myip=
if [ $? -eq 0 ]; then
  /sbin/ddns_custom_updated 1
else
  /sbin/ddns_custom_updated 0
fi
```