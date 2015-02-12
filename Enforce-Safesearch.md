By using dnsmasq, you can force all your LAN clients to use Google's Safesearch when doing any searches on their engines.

There are two different methods through which this can be done.

### Defining a different IP to www.google.com ###


First, we need to configure dnsmasq to alias www.google.com to forcesafesearch.google.com.  To do so, make sure JFFS + custom config/script options are enabled.  Then, create the following dnsmasq.conf.add script:

```
cname=www.google.com,forcesafesearch.google.com 
```

You could possibly also add your regionalized version of Google (i.e. www.google.ca).

Since dnsmasq cannot use upstream DNS to resolve the target hostname, it must be defined in your host file.  So, create a hosts.add custom config, with the following:

```
216.239.38.120 forcesafesearch.google.com
```

(you might want to first ensure that forcesafesearch.google.com resolves to the same IP, as it might possibly change based on your location)

After that, restart dnsmasq to activate your changes:

```
service restart_dnsmasq
```

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
