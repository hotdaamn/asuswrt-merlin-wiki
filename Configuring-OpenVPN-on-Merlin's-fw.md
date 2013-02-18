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
