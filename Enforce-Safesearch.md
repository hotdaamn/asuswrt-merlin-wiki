By using dnsmasq, you can force all your LAN clients to use Google's Safesearch when doing any searches on their engines.

For this, we need to configure dnsmasq to alias www.google.com to forcesafesearch.google.com.  To do so, make sure JFFS + custom config/script options are enabled.  Then, create a dnsmasq.conf.add script.

There are two different methods through which this can be done.

### Method 1: Defining a different IP to www.google.com ###

Put this in your dnsmasq.conf.add:
```
address=/www.google.com/216.239.38.120
```

Then, restart dnsmasq:

```
service restart_Dnsmasq
```

Alternatively:
### Method 2: Defining an alias to www.google.com ###

Put this in your dnsmasq.conf.add

```
cname=www.google.com,forcesafesearch.google.com 
```

Since dnsmasq cannot use upstream DNS to resolve the target hostname, it must be defined in your host file.  So, create a hosts.add custom config, with the following:

```
216.239.38.120 forcesafesearch.google.com
```

After that, restart dnsmasq to activate your changes:

```
service restart_dnsmasq
```

### Testing ###
Then try a nslookup from a computer.   www.google.com should now resolve to the forcesafesearch IP:

```
C:\Users\Merlin>nslookup www.google.com
Server:  router.asus.com
Address:  192.168.10.1

Name:    www.google.com
Addresses:  2607:f8b0:400b:806::1013
          216.239.38.120
```

Then, any Google search will force their Safe Search option to be enabled.

Note that you might want to check that forcesafesearch.google.com resolves to the same IP as mentionned in this article, as it might possibly change based on your location.

