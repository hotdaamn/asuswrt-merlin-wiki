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

Here is a working example, for afraid.org's free DDNS (you must update the URL to use your private API key from afraid.org). 

```
----- HTTPS -----                                                                                                           
#!/bin/sh    
KEY=xxxxxxxxREPLACE WITH KEYxxxxxxxxx                                                                                                                                                  
URL=/dynamic/update.php?$KEY                                                                                                                            
(echo "GET $URL"; sleep 5) | openssl s_client -quiet -connect freedns.afraid.org:443
if [ $? -eq 0 ]; then
    /sbin/ddns_custom_updated 1
else
    /sbin/ddns_custom_updated 0
fi

--- OR non-HTTPS --- (not recommended as your password is in plain hash).

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

### Google Domains

Transfer your domain to Google and enjoy free DDNS and other features.

```
#!/bin/sh

set -u

# args: username password hostname
google_dns_update() {             
  case $(curl -s https://$1:$2@domains.google.com/nic/update?hostname=$3) in
    good|nochg*) /sbin/ddns_custom_updated 1 ;;                             
    *) /sbin/ddns_custom_updated 0 ;;                                       
  esac                                                                      
}                                                                           
                                               
google_dns_update <username> <password> <subdomain>

```

### [DyNS](http://dyns.cx)

provide a number of free and premium DNS related services for home or office use.

```
#!/bin/sh
#
# http://dyns.cx/documentation/technical/protocol/v1.1.php
                
USERNAME=   
PASSWORD=   
HOSTNAME=
DOMAIN=  # optional                       
IP=${1}                                                                                                        
DEBUG= # set to true while testing                                                                                          
                                                                                                               
URL="http://www.dyns.net/postscript011.php?username=${USERNAME}&password=${PASSWORD}&host=${HOSTNAME}&ip=${IP}"
if [ -n "${DOMAIN}" ] ; then   
  URL="${URL}&domain=${DOMAIN}"
fi                         
if [ -n "${DEBUG}" ] ; then
  URL="${URL}&devel=1"     
fi                           
                             
wget -q -O - "$URL"          
if [ $? -eq 0 ]; then        
  /sbin/ddns_custom_updated 1
else                         
  /sbin/ddns_custom_updated 0
fi                           
```

### CloudFlare

If you use CloudFlare for your domains, this script can update any A record on your account.

```
#!/bin/sh

EMAIL= # Your CloudFlare E-mail address
ZONE= # The zone where your desired A record resides
RECORDID= # ID of the A record
RECORDNAME= # Name of the A record
API= # Your CloudFlare API Key
IP=${1}

curl -s --data "a=rec_edit&tkn=$API&email=$EMAIL&z=$ZONE&id=$RECORDID&type=A&name=$RECORDNAME&ttl=1&content=$IP" https://www.cloudflare.com/api_json.html
if [ $? -eq 0 ]; then
  /sbin/ddns_custom_updated 1
else
  /sbin/ddns_custom_updated 0
fi
```
