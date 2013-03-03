It is possible to generate your certificates on the router itself if you don't have access to a Linux machine, or if you don't have a Windows client installed with Easy-RSA.  Easy-RSA is a simple to use environment that is bundled with OpenVPN, and has been included in Asuswrt-Merlin.

### Setting up the environment
The first step is to initialize your work environment.  Ideally this should be done on a USB disk, but it can be done in /tmp (make sure you DO keep a copy of everything generated there, because it will be lost the next time you reboot the router!).  For this example, we will be using a USB disk mounted under /mnt/sda1.  First, copy the easy-rsa scripts by running the following command:

`setuprsa.sh /mnt/sda1`

This will create an easy-rsa folder on your USB disk, and copy all the required scripts there.  Then, enter that new directory:

`cd /tmp/sda1/easy-rsa`

Now step you will probably want to change the default values offered while generating the certificates.  Edit the file named "vars", either through the built-in "vi" editor (not recommended for novice users), or by installing the "nano" editor using Optware, or simply by copying the file edited on your computer.  The only fields you might want to change are these:

* export KEY_COUNTRY="US"
* export KEY_PROVINCE="CA"
* export KEY_CITY="SanFrancisco"
* export KEY_ORG="Fort-Funston"
* export KEY_EMAIL="me@myhost.mydomain"
* export KEY_EMAIL=mail@host.domain
* export KEY_CN=changeme
* export KEY_NAME=changeme
* export KEY_OU=changeme

You can also adjust the expiration date for keys if desired.  I do not recommend changing the expiration date for the CA, neither the key size - increasing from 1024 bytes will have a performance impact.

Once done, setup the environment by running the script this way:

`source ./vars`

There, initialize the environment:

`./clean-all`

Your environment is now ready to be used to generate your certificates.


### Generating the certificates
First, you need to generate your Certificate Authority (CA).  This will be the "master" key and certificate, which will be used to sign all client certificates, or revoke their access.  Make sure you store this in a safe, secure location (preferably NOT on the router itself!).  To generate the CA pair:

`./build-ca`

The Common Name (CN) is the most important field, as it will be what identifies your router.

Now, we need to build a router key/certificate pair:

`./build-key-server server1`

Use any name you want instead of "_server1_", but make sure that when asked for the Common Name that you enter the exact same name.  When asked to sign and to commit the new certificate, answer "y" to both questions.

Next, let's build one client key/certificate pair.  Same procedure (and once again pay attention to the Common Name, which must match the name you are specifying here instead of _client1_):

`./build-key client1`

You can create as many client keypairs as you need.  The CA file will be what determines which keys are allowed to connect.

One last thing - we need to generate the Diffie Hellman parameters (DH file), which is used to secure the key exchange between client and router.  Run the following command:

`./build-dh`

This operation can take a minute or two due to the low performance of the router's CPU (compared to a desktop).

All the generated files will now be located in the keys/ subdirectory.  Once again, make SURE you copy these in a safe location!  You now have all the required keys and certificates to configure your OpenVPN server.

If you need additional information, take a look at this excellent [tutorial](http://www.howtogeek.com/60774/connect-to-your-home-network-from-anywhere-with-openvpn-and-tomato/) designed for Tomato.


