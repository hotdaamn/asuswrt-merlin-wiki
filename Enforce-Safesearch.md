By using [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html), you can force all your LAN clients to use [Google's Safesearch](https://support.google.com/websearch/answer/186669) when doing any searches on their engines.

For this, we need to configure dnsmasq to alias `www.google.com` (and also perhaps all other [Google's domains](https://www.google.com/supported_domains)) to `forcesafesearch.google.com`.  To do so, make sure [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/Jffs) + [custom config/script](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts) options are enabled.  Then, create a `/jffs/configs/dnsmasq.conf.add` script.

There are two different methods through which this can be done.

### Method 1: Defining an alias to www.google.com ###

Put this in your `/jffs/configs/dnsmasq.conf.add`:

```
cname=www.google.com,forcesafesearch.google.com 
```

However, since dnsmasq cannot use upstream DNS to resolve the target hostname (i.e. forcesafesearch.google.com), it must be defined in your `hosts` file.  So, create also a `/jffs/configs/hosts.add` file with the following:

```
216.239.38.120 forcesafesearch.google.com
```

After that, restart dnsmasq to activate your changes:

```
service restart_dnsmasq
```

Alternatively:
### Method 2: Defining a different IP to www.google.com ###

Put this in your `/jffs/configs/dnsmasq.conf.add`:
```
address=/www.google.com/216.239.38.120
```

Then, restart dnsmasq:

```
service restart_dnsmasq
```

### Testing ###
Then try a `nslookup` from a computer in your LAN.   www.google.com should now resolve to the forcesafesearch's IP:

```
C:\Users\Merlin>nslookup www.google.com
Server:  router.asus.com
Address:  192.168.10.1

Name:    www.google.com
Addresses:  2607:f8b0:400b:806::1013
          216.239.38.120
```

Then, any Google search will force their Safe Search option to be enabled.

Note that you might want to check that forcesafesearch.google.com resolves to the same IP as mentioned in this article, as it might possibly change based on your location.
