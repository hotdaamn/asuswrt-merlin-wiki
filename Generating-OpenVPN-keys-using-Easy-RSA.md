(Draft)
It is possible to generate your certificates on the router itself if you don't have access to a Linux machine, or if you don't have a Windows client installed with Easy-RSA.  Easy-RSA is a simple to use environment that is bundled with OpenVPN, and has been included in Asuswrt-Merlin.

The first step is to initialize your work environment.  Ideally this should be done on a USB disk, but it can be done in /tmp (make sure you DO keep a copy of everything generated there, because it will be lost the next time you reboot the router!).  For this example, we will be using a USB disk mounted under /mnt/sda1.  First, copy the easy-rsa scripts by running the following command:

setuprsa.sh /mnt/sda1

This will create an easy-rsa folder on your USB disk, and copy all the required scripts there.  Then, enter that new directory:

cd /tmp/sda1/easy-rsa

There, initialize the environment:
./clean-all

Next step you will probably want to change the default values offered while generating the certificates.  Edit the file named "vars", either through the built-in "vi" editor (not recommended for novice users), or by installing the "nano" editor using Optware, or simply by copying the file edited on your computer.

Once done, setup the environment by running the script:
./vars

Your environment is now ready to be used to generate your certificates.

Generating the certificates
First, you need to generate your Certificate Authority (CA).  This will be the "master" certificate, which will be used to sign all client certificates, or revoke their access.  Make sure you store this in a safe, secure location (preferably NOT on the router itself!).  To generate the CA:
./build-ca
The Common Name is the most important field, as it will be what identifies your router.

Now, we need to build a router key/certificate pair:
./build-key server1
Use any name you want instead of "server1", but make sure that when asked for the Common Name that you enter the exact same name.  When asked to sign and to commit the new certificate, answer "y".

Next, let's build one client key/certificate pair.  Same procedure (and pay attention to the Common Name):
./build-key client1

One last thing - we need to generate the Diffie Hellman parameters (DH file), which is used to secure the key exchange between client and router.  Run the following command:
./build-dh

This operation can take a minute or two due to the low performance of the router's CPU (compared to a desktop).

