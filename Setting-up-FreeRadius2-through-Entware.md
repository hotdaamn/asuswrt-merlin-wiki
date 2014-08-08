## Introduction


In this tutorial we're gonna install freeradius2 (EAP-TTLS,PAP) from entware.
Why should I consider using WPA2-enterprise on my router ?
		
1.    Eliminates the security risks of PSK passwords 
2.    Puts a damper on snooping
3.    Enables enhanced security methods
4.    Entware's freeradius version works great on asus mips routers
5.    Get the best wireless security possible on your premium router

We have several authentication methods to choose from like EAP-TLS,EAP-PEAP, ... I think EAP-TTLS is one of the better options and it doesn't require client certs but still uses a secure tunnel (server certificates) before inner client authentication is taking place. 
Here are some more detailed descriptions:

* [Juniper](http://www.juniper.net/techpubs/software/aaa_802/sbrc/sbrc70/sw-sbrc-admin/html/EAP-024.html) 
* [Wikipedia](http://en.wikipedia.org/wiki/Extensible_Authentication_Protocol#EAP-TTLS)

The only disadvantage with EAP-TTLS is that there's no native support for it in Windows 7 and earlier but there's still an free third party solution for pc not sure about windows phone 8 if you cannot upgrade to 8.1. 

Instead of using RSA keys, I have used [elliptic curve](http://en.wikipedia.org/wiki/Elliptic_curve_cryptography) keys and this does work extremly well on freeradius and clients.

## Prerequisites 


A wired device with telnet,ssh for debugging and some basic linux skills recommended.

## Initial configuration


I will also assume that your storage device is already formatted as either Ext2 (USB stick) or Ext3 (External Harddrive). If not, look on the web for information on how to reformat your disk.

Initial steps:

* Enable the JFFS partition (as explained [here](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS))
* Setup Entware (as explained [here](https://github.com/RMerl/asuswrt-merlin/wiki/Entware))

## Installation

1) Install required packages

```
opkg install freeradius2 \
freeradius2-common \
freeradius2-mod-eap \
freeradius2-mod-eap-tls \
freeradius2-mod-eap-ttls \
freeradius2-mod-files \
freeradius-mod-pap
```

2) I have created a very stripped version of the needed config files & missing one after testing. For making things a bit easier we're gonna use that one.

```
cd /opt/etc/ && cp -r freeradius2 freeradius2.bak
cd freeradius2 && rm -rf *
opkg install wget
/opt/bin/wget --no-check-certificate -O - http://goo.gl/Ykov6H | tar -xzC /opt/etc/freeradius2
```

3) Generating certs

```
cd /opt/etc/freeradius2/certs
/opt/bin/wget --no-check-certificate http://goo.gl/kNg6Ms -O openssl.cnf
mkdir -p CA/private new export
```
Private Key & CA Cert
When generating the CA Certificate Request do not leave fields blank, we can use that as an extra check of the validity of the certificate & enter a password to protect the privatekey
Example of mine: C=BE, ST=Limburg, L=Hoeselt, O=Last-Name, OU=IT Department, CN=Certificate Authority Last-Name

```
openssl ecparam -name secp521r1 -genkey -noout | openssl ec -aes256 -out CA/private/ec-cakey.pem
openssl req -new -x509 -out CA/ec-cacert.pem -outform PEM -SHA512 -key CA/private/ec-cakey.pem \
 -keyform PEM  -days 3650 -extensions v3_ca -config ./openssl.cnf
```

Export PEM to DER (So we can import the key on Windows machines)

```
openssl x509 -outform DER -in CA/ec-cacert.pem -out export/ec-cacert.der
```

Export ec-cacert.pem to p12 (So we can install the certificate on android phones)
You will be asked for the private key password earlier and to create an export password. You will need the export password when installing the certificate on your phone.

```
openssl pkcs12 -export -in CA/ec-cacert.pem -inkey CA/private/ec-cakey.pem \
 -out export/ec-cacert.p12 -name "Name of Your Certificate" -cacerts
```

Server
When generating Server Certificate Request do not enter an challenge password & optional company name
Example of mine: C=BE, ST=Limburg, L=Hoeselt, O=Last-Name, OU=IT Department, CN=Server

```
openssl req -nodes -SHA512 -newkey ec:CA/ec-cacert.pem -new -days 3650 \ 
-out new/ec-server_req.pem -keyout new/ec-server_key.pem -config ./openssl.cnf
```

Encrypt server key en generate server certificate

```
mv new/ec-server_key.pem new/ec-server_key_temp.pem && openssl ec \ 
-in new/ec-server_key_temp.pem -aes256 -out new/ec-server_key.pem && \
rm -rf new/ec-server_key_temp.pem
openssl x509 -req -extfile openssl.cnf -out new/ec-server_cert.pem -SHA512 \
 -CA CA/ec-cacert.pem -CAkey CA/private/ec-cakey.pem -in new/ec-server_req.pem \
 -days 3650 -set_serial 0x01
```

Move ec-server_cert.pem  ec-server_key.pem to certs directory and remove new directory

```
mv new/ec-server_cert.pem  new/ec-server_key.pem ./ && rm -rf new
chmod 0400 ec-server_cert.pem && chmod 0400 ec-server_key.pem
```

Remove openssl.cnf

```
rm openssl.cnf
```

Generate Diffie Hellman

```
openssl dhparam -check -text -2 2048 -out dh (takes some time on router)
chmod 0600 dh
```

Move & Store CA & export dirs on a secure place. You will need a samba share or scp or winscp to copy them to your pc. 

You will need: 
* ec-cacert.pem on linux as CA certificate 
* ec-cacert.der or .p12 on Windows as CA certificate
* ec-cacert.p12 on android -> place it on the root of a sdcard

Now you should only have ec-server_cert.pem, ec-server_key.pem and dh in your certs directory

4) Edit eap.conf (/opt/etc/freeradius2/eap.conf)

```
cd /opt/etc/freeradius2 && vi eap.conf
```

Change private key pass to the password you have used to encrypt the ec-server_key.pem
Open an other terminal and go to the certs directory.

```
cd /opt/etc/freeradius2/certs
openssl x509 -noout -in ec-server_cert.pem -issuer
```

Replace this DN with the one at check_cert_issuer

5) Edit clients.conf (/opt/etc/freeradius2/clients.conf)

Change network range, mask at client line. I'm assuming that the IP-address of the router = 192.168.1.1 if this is not the case change it here and also change it /opt/etc/freeradius2/radiusd.conf & /opt/etc/freeradius2/sites/inner-tunnel. For the secret AP I recommended a 32 chars key. It is possible to use longer keys then 32 chars but I found out that the WiFi became slower. (copy this to a text editor for later if you're already using on a wired device)
Use a site like [this one](http://goo.gl/Muhc) to generate the key and do not use strange punctations, etc ... I'm not sure which one are allowed and which one not.

6) Edit users file (/opt/etc/freeradius2/users)

Change the login name and password to the one you are going to use.
You also can add, change attributes like session-timeout to a more appropriate value for you.

7) Edit radiusd.conf (/opt/etc/freeradius2/radiusd.conf)

We are going to change the max request value dependent on the number of wireless clients that are going to connect with our radius server.

max_requests: The maximum number of requests which the server keeps
track of.  This should be 256 multiplied by the number of clients.
e.g. With 4 clients, this number should be 1024.

8) Let's run a test to check if the server initialized correctly

```
radiusd -XX
```

If the server is listening for requests you're good to otherwise check error(s)
Start the server

```
/opt/etc/init.d/S55radiusd start
```

9) Configure router wireless settings for WPA2-Enterprise

Use the wired device now if you are not using it and login to the routers webinterface and go to the wireless page, change authentication method to WPA2 enterprise for 2.4Ghz and 5Ghz if you use that band also.

Go to the "Radius Setting" tab and change the IP-address of the radius server to the IP-address of your router.
Change port to 1812
Enter the secret from the clients.conf which we still have open in a texteditor or open clients.conf in a telnet, ssh session to copy, paste it.
Do the same thing for the other band if you are using it.
At this moment server should be running successfully and being able to receive authentication request.

## Configuring wireless clients


On Windows 7, 8 we need first to import the ec-cacert.der.
Place ec-cacert.der on a USB stick or something portable that you could copy the cert over to the wireless clients.

```
1. Open mmc as administrator -> File Add/Remove snap-in -> certificates -> add 
   -> computer account -> local computer -> finish -> ok 
2. Open certificates, Open Trusted Root Certification Authorities.
3. Right click on certificates -> all task -> import -> next -> browse to ec-cacert.der , ...
4. Close console1 do not save settings.
```
Windows 8 users can skip next step.
For Windows 7 there's an old alternative called SecureW2  which was free until version 1.13. But there's even a better alternative which does works correctly with ec keys. It's eap-ttls software especially for intel wireless cards but it does also works for broadcom devices. Assuming all of you Windows 7 users also have an "intel" card here's [the driver link](http://goo.gl/tqbfz4)

Now we need to create the network

```
Go to Network and sharing server -> manage wireless networks
Add Manually create network profile -> Network name your ssid -> security type WPA-enterprise AES -> next -> change connection settings 
-> security tab -> (for windows 8 eap-ttls windows 7 use intel-eap-ttls) -> settings
```

Windows 7

``` 
-select PAP
-Username
-Domain leave blank
-Password 
-Roaming identity anonymous

Check Validate Server Certificate
	-Certificate Issuer (Select your certificate)

Check Specify Server or Certificate Name
	-Server name must match the specified entry exactly (Your server CN)
```

Windows 8

[Configure EAP-TTLS](http://adamsync.wordpress.com/2012/05/08/eap-ttls-on-windows-2012-build-8250/#comments)

Android Devices

```
For android place the .p12 file on your sdcard.
On my phone 4.1.1, to install a certificate go to security -> install from storage.
You will be asked for the export password enter it if you have and install the certificate.  When connecting to the network	
Choose EAP-TTLS as eap-method	
Choose PAP as Phase 2 verification
Identity 
Password 
Enable advanced options
CA certificate the one you installed
Anonymous identity just enter anonymous.
That's it you should be able to connect successfully.
```

iOS Devices

* Install certificate on [iOS device](https://support.quovadisglobal.com/KB/a64/how-do-i-install-digital-certificate-onto-an-iphone-ipad.aspx)

* [iOS Support](http://support.apple.com/kb/DL1465) for EAP-TTLS, PAP.
