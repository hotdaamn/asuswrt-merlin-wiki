A VPN is a Virtual Private Network. Using a public internet network, it is possible to virtualy built a private network between a server somewhere and a client, usually your PC, but may be any intelligent devices. The VPN circuitry will garanty you that the communication between you and the accessed server cannot be understood by someone else.

There is many products distributed to build such a virtual channel. One of them is free and supported by the community: OpenVPN
All the information about it is gathered on http://openvpn.net/

As you can imagine, there is two ends: 

	* the server 
	* the clients.

The OpenVPN server is already installed in Merlin's version of the ASUS firmware. You can access it throught the "VPN server" choice in the "Advanced settings". Two VPN servers are offered on that page:

	* the one coming with the ASUS firmware: PPTP. 
	* the one added by Merlin (probably coming from the project/firmware TOMATO)

OpenVPN is supposed to be a lot more efficient, flexible and secure than the PPTP one.

Since the OpenVPN server is already included, you don't have to install it to setup a secure path with your ASUS router. That said, you have to:

	* generate all the keys and certificates required to create and maintain a secure channel on both ends: server and clients
	* use these keys and certificates to configure the ASUS OpenVPN server
	* install an OpenVPN client on the devices requiring an access to the OpenVPN server
	* use keys and certificates to configure the clients.

The community page is located at http://openvpn.net/index.php/download/community-downloads.html
At the time of this writing, the last version (released on 2013.01.08) is 2.3.0
From that page you can download the Windows installer. 

Setting up your own Certificate Authority (CA) and generating certificates and keys for an OpenVPN server and multiple clients

Overview

The first step in building an OpenVPN 2.0 configuration is to establish a PKI (public key infrastructure). The PKI consists of:

a separate certificate (also known as a public key) and private key for the server and each client, and
a master Certificate Authority (CA) certificate and key which is used to sign each of the server and client certificates.
OpenVPN supports bidirectional authentication based on certificates, meaning that the client must authenticate the server certificate and the server must authenticate the client certificate before mutual trust is established.

Both server and client will authenticate the other by first verifying that the presented certificate was signed by the master certificate authority (CA), and then by testing information in the now-authenticated certificate header, such as the certificate common name or certificate type (client or server).

This security model has a number of desirable features from the VPN perspective:

The server only needs its own certificate/key -- it doesn't need to know the individual certificates of every client which might possibly connect to it.
The server will only accept clients whose certificates were signed by the master CA certificate (which we will generate below). And because the server can perform this signature verification without needing access to the CA private key itself, it is possible for the CA key (the most sensitive key in the entire PKI) to reside on a completely different machine, even one without a network connection.
If a private key is compromised, it can be disabled by adding its certificate to a CRL (certificate revocation list). The CRL allows compromised certificates to be selectively rejected without requiring that the entire PKI be rebuilt.
The server can enforce client-specific access rights based on embedded certificate fields, such as the Common Name.
Note that the server and client clocks need to be roughly in sync or certificates might not work properly.
Generate the master Certificate Authority (CA) certificate & key

In this section we will generate a master CA certificate/key, a server certificate/key, and certificates/keys for 3 separate clients.

For PKI management, we will use a set of scripts bundled with OpenVPN.

If you are using Linux, BSD, or a unix-like OS, open a shell and cd to the easy-rsa subdirectory of the OpenVPN distribution. If you installed OpenVPN from an RPM file, the easy-rsa directory can usually be found in /usr/share/doc/packages/openvpn or /usr/share/doc/openvpn-2.0 (it's best to copy this directory to another location such as /etc/openvpn, before any edits, so that future OpenVPN package upgrades won't overwrite your modifications). If you installed from a .tar.gz file, the easy-rsa directory will be in the top level directory of the expanded source tree.

If you are using Windows, open up a Command Prompt window and cd to \Program Files\OpenVPN\easy-rsa. Run the following batch file to copy configuration files into place (this will overwrite any preexisting vars.bat and openssl.cnf files):

init-config
Now edit the vars file (called vars.bat on Windows) and set the KEY_COUNTRY, KEY_PROVINCE, KEY_CITY, KEY_ORG, and KEY_EMAIL parameters. Don't leave any of these parameters blank.

Next, initialize the PKI. On Linux/BSD/Unix:

. ./vars
./clean-all
./build-ca
On Windows:

vars
clean-all
build-ca
The final command (build-ca) will build the certificate authority (CA) certificate and key by invoking the interactive openssl command:

ai:easy-rsa # ./build-ca
Generating a 1024 bit RSA private key
............++++++
...........++++++
writing new private key to 'ca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [KG]:
State or Province Name (full name) [NA]:
Locality Name (eg, city) [BISHKEK]:
Organization Name (eg, company) [OpenVPN-TEST]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:OpenVPN-CA
Email Address [me@myhost.mydomain]:
Note that in the above sequence, most queried parameters were defaulted to the values set in the vars or vars.bat files. The only parameter which must be explicitly entered is the Common Name. In the example above, I used "OpenVPN-CA".

Generate certificate & key for server

Next, we will generate a certificate and private key for the server. On Linux/BSD/Unix:

./build-key-server server
On Windows:

build-key-server server
As in the previous step, most parameters can be defaulted. When the Common Name is queried, enter "server". Two other queries require positive responses, "Sign the certificate? [y/n]" and "1 out of 1 certificate requests certified, commit? [y/n]".

Generate certificates & keys for 3 clients

Generating client certificates is very similar to the previous step. On Linux/BSD/Unix:

./build-key client1
./build-key client2
./build-key client3
On Windows:

build-key client1
build-key client2
build-key client3
If you would like to password-protect your client keys, substitute the build-key-pass script.

Remember that for each client, make sure to type the appropriate Common Name when prompted, i.e. "client1", "client2", or "client3". Always use a unique common name for each client.

Generate Diffie Hellman parameters

Diffie Hellman parameters must be generated for the OpenVPN server. On Linux/BSD/Unix:

./build-dh
On Windows:

build-dh
Output:

ai:easy-rsa # ./build-dh
Generating DH parameters, 1024 bit long safe prime, generator 2
This is going to take a long time
.................+...........................................
...................+.............+.................+.........
......................................
Key Files

Now we will find our newly-generated keys and certificates in the keys subdirectory. Here is an explanation of the relevant files:

Filename	Needed By	Purpose	Secret
ca.crt	server + all clients	Root CA certificate	NO
ca.key	key signing machine only	Root CA key	YES
dh{n}.pem	server only	Diffie Hellman parameters	NO
server.crt	server only	Server Certificate	NO
server.key	server only	Server Key	YES
client1.crt	client1 only	Client1 Certificate	NO
client1.key	client1 only	Client1 Key	YES
client2.crt	client2 only	Client2 Certificate	NO
client2.key	client2 only	Client2 Key	YES
client3.crt	client3 only	Client3 Certificate	NO
client3.key	client3 only	Client3 Key	YES
The final step in the key generation process is to copy all files to the machines which need them, taking care to copy secret files over a secure channel.

Now wait, you may say. Shouldn't it be possible to set up the PKI without a pre-existing secure channel?

 

The answer is ostensibly yes. In the example above, for the sake of brevity, we generated all private keys in the same place. With a bit more effort, we could have done this differently. For example, instead of generating the client certificate and keys on the server, we could have had the client generate its own private key locally, and then submit a Certificate Signing Request (CSR) to the key-signing machine. In turn, the key-signing machine could have processed the CSR and returned a signed certificate to the client. This could have been done without ever requiring that a secret .key file leave the hard drive of the machine on which it was generated.

Creating configuration files for server and clients

Getting the sample config files

It's best to use the OpenVPN sample configuration files as a starting point for your own configuration. These files can also be found in

the sample-config-files directory of the OpenVPN source distribution
the sample-config-files directory in /usr/share/doc/packages/openvpn or /usr/share/doc/openvpn-2.0 if you installed from an RPM package
Start Menu -> All Programs -> OpenVPN -> OpenVPN Sample Configuration Files on Windows
Note that on Linux, BSD, or unix-like OSes, the sample configuration files are named server.conf and client.conf. On Windows they are named server.ovpn and client.ovpn.

Editing the server configuration file

The sample server configuration file is an ideal starting point for an OpenVPN server configuration. It will create a VPN using a virtual TUN network interface (for routing), will listen for client connections on UDP port 1194 (OpenVPN's official port number), and distribute virtual addresses to connecting clients from the 10.8.0.0/24 subnet.

Before you use the sample configuration file, you should first edit the ca, cert, key, and dh parameters to point to the files you generated in the PKI section above.

At this point, the server configuration file is usable, however you still might want to customize it further:

If you are using Ethernet bridging, you must use server-bridge and dev tap instead of server and dev tun.
If you want your OpenVPN server to listen on a TCP port instead of a UDP port, use proto tcp instead of proto udp (If you want OpenVPN to listen on both a UDP and TCP port, you must run two separate OpenVPN instances).
If you want to use a virtual IP address range other than 10.8.0.0/24, you should modify the server directive. Remember that this virtual IP address range should be a private range which is currently unused on your network.
Uncomment out the client-to-client directive if you would like connecting clients to be able to reach each other over the VPN. By default, clients will only be able to reach the server.
If you are using Linux, BSD, or a Unix-like OS, you can improve security by uncommenting out the user nobody and group nobody directives.
If you want to run multiple OpenVPN instances on the same machine, each using a different configuration file, it is possible if you:

Use a different port number for each instance (the UDP and TCP protocols use different port spaces so you can run one daemon listening on UDP-1194 and another on TCP-1194).
If you are using Windows, each OpenVPN configuration needs to have its own TAP-Win32 adapter. You can add additional adapters by going to Start Menu -> All Programs -> OpenVPN -> Add a new TAP-Win32 virtual ethernet adapter.
If you are running multiple OpenVPN instances out of the same directory, make sure to edit directives which create output files so that multiple instances do not overwrite each other's output files. These directives include log, log-append, status, and ifconfig-pool-persist.
Editing the client configuration files

The sample client configuration file (client.conf on Linux/BSD/Unix or client.ovpn on Windows) mirrors the default directives set in the sample server configuration file.

Like the server configuration file, first edit the ca, cert, and key parameters to point to the files you generated in the PKI section above. Note that each client should have its own cert/key pair. Only the ca file is universal across the OpenVPN server and all clients.
 

Next, edit the remote directive to point to the hostname/IP address and port number of the OpenVPN server (if your OpenVPN server will be running on a single-NIC machine behind a firewall/NAT-gateway, use the public IP address of the gateway, and a port number which you have configured the gateway to forward to the OpenVPN server).
 

Finally, ensure that the client configuration file is consistent with the directives used in the server configuration. The major thing to check for is that the dev (tun or tap) and proto (udp or tcp) directives are consistent. Also make sure that comp-lzo and fragment, if used, are present in both client and server config files.
 