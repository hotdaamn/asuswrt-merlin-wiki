Asuswrt-Merlin's OpenVPN interface tries to reproduce as closely as possible the interface used on the Tomato firmware (which shares the same OpenVPN code).  Look around for tutorials written for Tomato, those can easily be applied to this firmware as well.

One such tutorial that is recommended (for configuring the OpenVPN server) can be found [here](http://www.howtogeek.com/60774/connect-to-your-home-network-from-anywhere-with-openvpn-and-tomato/).

### Custom client config files

Starting with 3.0.0.4.372.31, you can now provide the OpenVPN server with your own custom client configuration (the CCD files).  First, make sure you do have [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) enabled.

Enable the "Manage Client-Specific Options" option under OpenVPN Server.  Next, create the following directory (use ccd2 if you are providing config files for the second OpenVPN Server instance):

```
/jffs/configs/openvpn/ccd1/
```
In this directory put the client config files you wish to provide your OpenVPN server with, each file being named after the common name of the targeted client.

Restart the OpenVPN server to have the files applied to the configuration.

For more info about CCD files, please see the official OpenVPN documentation.
