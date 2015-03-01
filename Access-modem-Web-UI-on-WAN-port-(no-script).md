This guide provides an easy configuration to access web-based UI of your xDSL/Cable modem connected to the wan port of your Asus router. **Important! - This only works if you are able to acess the modem's configuration (some ISP have restrictions on their equipment).** For this to work and avoid issues, Modem and Router will be configured on different subnets.

![What we will do.](http://s22.postimg.org/s5vwfy80h/untitled.png)

**Step 1. Change your Modem ip to** _192.168.0.1_ **(Below an inlustrated example with a TP-Link modem)**

![Modem IP](http://s8.postimg.org/qg66vow8l/Captura_47.png)

**Step 2. Go to _LAN -> LAN IP_. Make sure your Router is on a different subnet, by default It is. Click** _Apply_.**(Default ip adress will be 192.168.1.1)**

![Router IP](http://s17.postimg.org/dqeydphhb/Captura_48.png)

**Step 3. Go to WAN -> Internet Connection -> WAN IP Setting (section). Assign your Router an IP from the Modem's subnet:** **Switch** _Get the WAN IP automatically_ **to** _No_ **and set** _IP = 192.168.0.2 | Subnet Mask = 255.255.255.0 | Gateway = 192.168.0.1_ **and click** _Apply_ **at the bottom.**

![WAN IP](http://s30.postimg.org/vgi76j38h/Captura_49.png)

![](http://s14.postimg.org/fbq9xcunh/Captura_48.png)

**Step 4. Test if you can Access the modem's interface. Open your browser and enter** _192.168.0.1_

![](http://s3.postimg.org/jzg8wsimr/Captura_50.png)

**That's it !**