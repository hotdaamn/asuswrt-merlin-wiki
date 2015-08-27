### Introduction
If you set the DDNS (dynamic DNS) service to "Custom", then you can fully control the update process through a `ddns-start` user script (which could launch a custom update client, or run a simple "wget" on a provider's update URL). The ddns-start script is passed the WAN IP as an argument.

Note that your custom script is responsible for notifying the firmware on the success or failure of the process.  To do this your script must execute:

```
/sbin/ddns_custom_updated 0 or 1
```
(where 0 = failure, 1 = successful update)

If you can't determine the success or failure, then report it as a success to ensure that the firmware won't continuously try to force an update.

Finally, like all [[user scripts|User-scripts]], the option to support custom scripts and config files must be enabled under Administration -> System.

After enabling custom scripts, place the contents of your update script in `/jffs/scripts/ddns-start` 


# Sample Scripts


### afraid.org

Here is a working example, for afraid.org's free DDNS (you must update the URL to use your private API key from afraid.org). 

```
----- HTTPS -----                                                                                                           
#!/bin/sh

curl -k "https://freedns.afraid.org/dynamic/update.php?PASTE_YOUR_KEY_HERE" >/dev/null

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

### [dnsExit.com](http://www.dnsexit.com/Direct.sv?cmd=dynDns)
* **_NOTE: the example below uses non-HTTPS which isn't recommended.  dnsExit.com doesn't have HTTPS method available._**

Free DNS server that also offers DDNS services.
```
#!/bin/sh
USER=
PASS=
DOMAIN=
wget -qO - "http://update.dnsexit.com/RemoteUpdate.sv?login=$USER&password=$PASS&host=$DOMAIN"
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

U=xxxx
P=xxxx
H=xxxx

# args: username password hostname
google_dns_update() {
  CMD=$(curl -s https://$1:$2@domains.google.com/nic/update?hostname=$3)
  logger "google-ddns-updated: $CMD"
  case "$CMD" in
    good*|nochg*) /sbin/ddns_custom_updated 1 ;;
    abuse) /sbin/ddns_custom_updated 1 ;;
    *) /sbin/ddns_custom_updated 0 ;;
  esac
}

google_dns_update $U $P $H

exit 0

```

### [DyNS](http://dyns.cx)
* **_NOTE: the example below uses non-HTTPS which isn't recommended.  See example for afraid above._**

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

curl -fs -o /dev/null -X PUT "https://api.cloudflare.com/client/v4/zones/$ZONE/dns_records/$RECORDID" \
	-H "X-Auth-Email: $EMAIL" \
	-H "X-Auth-Key: $API" \
	-H "Content-Type: application/json" \
	--data '{"id":"'$RECORDID'","type":"A","name":"'$RECORDNAME'","content":"'$IP'"}'

if [ $? -eq 0 ]; then
	/sbin/ddns_custom_updated 1
else
	/sbin/ddns_custom_updated 0
fi
```

### [pdd.yandex.ru](https://domain.yandex.com)
If you use domain.yandex.com for your domains, this script can update any A/AAAA record on your account. Replace `router.yourdomain.com`, `token` and `id` with your own values.
```
# Get token at https://pddimp.yandex.ru/token/index.xml?domain=yourdomain.com
token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Get record ID from https://pddimp.yandex.ru/nsapi/get_domain_records.xml?token=$token&domain=yourdomain.com
# <record domain="router.yourdomain.com" priority="" ttl="21600" subdomain="home" type="A" id="yyyyyyyy">...</record>
id=yyyyyyyy

/usr/sbin/curl --silent "https://pddimp.yandex.ru/nsapi/edit_a_record.xml?token=$token&domain=yourdomain.com&subdomain=home&record_id=$id&ttl=900&content=router.yourdomain.com" > /dev/null 2>&1
if [ $? -eq 0 ];
then
    /sbin/ddns_custom_updated 1
else
    /sbin/ddns_custom_updated 0
fi
```

### [Namecheap](https://www.namecheap.com)
If you use Namecheap for your domains, this script can update any A record on your account. The script is currently (as of Aug 1 2015) required because the built-in script uses HTTP, while Namecheap requires HTTPS. To use this, replace `HOSTNAME`, `DOMAIN` and `PASSWORD` with your own values. You can refer to the [DDNS FAQ at Namecheap](https://www.namecheap.com/support/knowledgebase/article.aspx/36/11/how-do-i-start-using-dynamic-dns) for steps required. 
```
#!/bin/sh
# Update the following variables:
HOSTNAME=[hostname]
DOMAIN=[domain.com]
PASSWORD=XXXXXXXXXXXXXXXXXXXXXXXX

# Should be no need to modify anything beyond this point
/usr/sbin/wget -qO - "https://dynamicdns.park-your-domain.com/update?host=$HOSTNAME&domain=$DOMAIN&password=$PASSWORD&ip="
if [ $? -eq 0 ]; then
  /sbin/ddns_custom_updated 1
else
  /sbin/ddns_custom_updated 0
fi
```

### [DNS-O-Matic](https://www.dnsomatic.com)
If you use DNS-O-Matic to update your domains, this script can update all or a single host record on your account.  To use this, replace `dnsomatic_username`, `dnsomatic_password` with your own values. You can refer to the [DNS-O-Matic API Documentation](https://www.dnsomatic.com/wiki/api#sample_updates) for additional info.

Note: the HOSTNAME specified in the script below will update all records setup in your DNS-O-Matic account to have it only update a single host you will need to modify it accordingly.  In some cases this may require you to specify the host entry, sometimes the domain entry.  

To troubleshoot update issues you can run the curl command directly from the command line by passing in your details and removing the --silent option.  If you get back good and your IP address back you've got it setup correctly.  If you get back nohost, you're not passing in the correct hostname value.
 
```
#!/bin/sh
# Update the following variables:
USERNAME=dnsomatic_username
PASSWORD=dnsomatic_password
HOSTNAME=all.dnsomatic.com

# Should be no need to modify anything beyond this point
/usr/sbin/curl -k --silent "https://$USERNAME:$PASSWORD@updates.dnsomatic.com/nic/update?hostname=$HOSTNAME&wildcard=NOCHG&mx=NOCHG&backmx=NOCHG&myip=" > /dev/null 
if [ $? -eq 0 ]; then
  /sbin/ddns_custom_updated 1
else
  /sbin/ddns_custom_updated 0
fi
```
