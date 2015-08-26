## Introduction ##

This guide will help you program which devices go through the [VPN](http://en.wikipedia.org/wiki/Virtual_private_network) and which devices go to your local [ISP](http://en.wikipedia.org/wiki/Internet_service_provider) by creating 2 different scripts.
Furthermore It will also show you how to Block VPN devices when VPN is down, but still allow non-VPN traffic.
This guide is specifically tailored for ASUS routers running Merlin Firmware.

## Prerequisites ##

* An Asuswrt-Merlin compatible ARM-based router with Asuswrt-Merlin installed (fwmarks on MIPS are broken)
* [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) partion enabled and formatted
* A working VPN on your router (tested manually making sure the VPN works)
* A [winSCP](http://winscp.net/eng/download.php#download2) client setup with SSH enabled on your ROUTER
* [Notepad ++](http://notepad-plus-plus.org/)

## Installation ##

STEP A:

1. Make sure [VPN](https://torguard.net/knowledgebase.php?action=displayarticle&id=148) is ON and Start with WAN option
2. Goto to Administration > System
3. Enable JFFS partition - YES
4. Format JFFS partition at next boot - YES
5. Reboot Router

STEP B:

This script by DEFAULT routes all devices through Local ISP.

All you need to do is add the IP addresses of each device you want to pass through the VPN one line at a time at the end of the script. Dead simple :)

STEP C:

Normally the router will assign IP addresses on the fly and the IP addresses listed above can potentially change. If you want to ensure this does not happen, do the following modifications to your router:

1. Under “Advanced Settings” on the left bar click “LAN”.
2. Click the “DHCP Server” tab.
3. Under “Manually Assigned IP around the DHCP list (Max Limit : 128)” you can assign fixed IP addresses to the devices you want to include.
4. Click the red arrow next to the empty window labeled “MAC Address” and you should see a list of all your connected devices.
5. Choose the device you want to include to the VPN. Click the plus arrow.
6. Repeat until you have selected all the devices you want to include to the VPN.
7. Now these devices will have permanent IP addresses, and you do not have to worry that they may change and mess up your script in the future.

STEP D:

Next you need to customize the script by telling it which devices will pass through the VPN. 
You need to repeat the following line for each device you want to let pass through the VPN which is at the very end of the script. 
In the script I have added 2 lines which would be for 2 separate devices. 
If you have more devices you will need to add a line for each device with their corresponding IP address:

each device has the following command:

iptables -t mangle -A PREROUTING -i br0 -m iprange --src-range 192.168.x.xxx -j MARK --set-mark 0

All you need to do is replace “192.168.1.xxx” with the actual IP address of each of the devices you want to include to the VPN.

Copy the script below to your notepad ++ and make sure you change xx with your devices values.
Now save the script as openvpn-event with no extension!

    #!/bin/sh
    
     Sleep 2
     
    for i in /proc/sys/net/ipv4/conf/*/rp_filter ; do
      echo 0 > $i
    done

    ip route flush table 100
    ip route del default table 100
    ip rule del fwmark 1 table 100
    ip route flush cache
    iptables -t mangle -F PREROUTING

    ip route show table main | grep -Ev ^default | grep -Ev tun11\
      | while read ROUTE ; do
          ip route add table 100 $ROUTE
    done

    ip route add default table 100 via $(nvram get wan_gateway)
    ip rule add fwmark 1 table 100
    ip route flush cache

    iptables -t mangle -A PREROUTING -i br0 -j MARK --set-mark 1

    iptables -t mangle -A PREROUTING -i br0 -m iprange --src-range 192.168.x.xxx -j MARK --set-mark 0

    exit 1

STEP E:

Now you are ready to add a firewall script which will determine which devices will stop traffic if the VPN drops connection, This will not affect traffic to devices on local ISP.
When the VPN is back up the devices will resume internet connectivity.

Please copy and paste the script below to notepad ++ and edit the IP address XX.X with the values for each device on the VPN which should be the same as in the openvpn-event script you created earlier, and make sure the tunnel points to the proper VPN switch that you are using on your Router.
For example; tun11 is VPN 1 and tun12 is VPN 2 on your router.

Now save as firewall-start without any extensions.

    #!/bin/sh

    sleep 4
        
    iptables -I FORWARD -i br0 -o tun11 -j ACCEPT
    iptables -I FORWARD -i tun11 -o br0 -j ACCEPT
    iptables -I FORWARD ! -o tun11 -s 192.168.xx.xxx -j DROP
    iptables -I FORWARD ! -o tun11 -s 192.168.xx.xxx -j DROP
    iptables -I INPUT -i tun11 -j REJECT
    iptables -t nat -A POSTROUTING -o tun11 -j MASQUERADE
    
    chmod a+rx /jffs/scripts/firewall-start

STEP F:

Now you need to log on to your router using winSCP.
You will need to have SSH enabled on your router.
To enable SSH on Merlin do this:

1. Under “Advanced Settings” on the left bar click “Administration”.
2. Under “Miscellaneous” make sure these are set as follows:
 
 Enable SSH - Yes

 Allow SSH Port Forwarding - No

 SSH service port - 22

 Allow SSH access from WAN - No

 Allow SSH password login - Yes

 Enable SSH Brute Force Protection - No

3. After you made the changes hit apply.


STEP G:

Please read Final Notes before copying scripts to the router.

You are now ready to copy the scripts you created to the router: 

1. Download [winSCP](http://winscp.net/eng/download.php#download2)
2. Install WinSCP then start it up.
3. To make a new connection fill in the fields as follows:
File Protocol – SCP (NOTE: MAKE SURE IT IS SET TO SCP)
Hostname: 192.168.xx.x replace x with values of your routers IP address
Username/Password: Whatever you use to login to the router
Port 22
4. Save login settings and then hit Login.
5. This will log you into your router. There will be two folder trees. On the right is your routers folder tree.
6. In the router folder tree, you need to go up to the root folder, where you will see the jffs folder under the root.
7. double click on the JFFS Folder then double click the Scripts Folder.
8. Navigate on the left folder tree and find the scripts you created. Drag the openvpn-event to the this folder (/jffs/scripts). test script and then copy firewall-start file
9. Once the file is copied, right click > Properties > Change Octal to 0777
10. Close WinSCP.
11. Reboot Router.

##Final Notes##

I have tried these scripts on AC66,68 and 87 routers and the way work fine.

First copy the openvpn-event script and make sure it works before you copy the next firewall-start script over to the router.
If you experience any problems change the sleep time from +/- intervals of 1.
Make sure you reboot each time you make a change and be sure to test if it works properly.
Once the first script loads and works you are ready to copy the second script firewall-start to the router. As you did with the previous script increase or decrease the sleep time by intervals of 1 and reboot until both scripts work properly.

If all worked well you should have all devices connecting to local ISP and the selected devices will pass through the VPN. 
Use [ipchicken](http://www.ipchicken.com) to check the IP address of your devices making sure that the devices that are on the VPN show the VPN address and the Devices that are not on the VPN should show your ISP address.
Now Test the firewall script by stopping the VPN on the router and see if the traffic stops from the selected VPN devices.
At this point all traffic should stop working on the VPN but traffic will still work on the non VPN devices.
If that worked then turn the VPN back on. The selected VPN devices should now be back up and running :)
You should also notice that the traffic passing via your ISP never got affected.

## VPN or Local ISP batch file for Windows ##

I have created a batch file for windows that you can switch from VPN to Local ISP on the fly without rebooting your pc. This script works with win7 and win8

Simply copy and paste the script below to notepad and save it as VPN.bat in your documents folder.
right click on VPN.bat and create a shortcut. now right click on the shortcut and click on properties then advance and enable run as admin. Move the shortcut to your desktop.

Note: change "Ethernet" to whatever name you have given to your network card. 
You can find out the name in Control Panel\Network and Internet\Network Connections take a look at the adapter name that your computer is using. Also use IP address that corresponds to the IP range you created for your VPN Scripts.

    @echo off
    echo Choose: 
    echo [A] VPN 
    echo [B] Local ISP
    echo. 

    :choice 
    SET /P C=[A,B]? 
    for %%? in (A) do if /I "%C%"=="%%?" goto A 
    for %%? in (B) do if /I "%C%"=="%%?" goto B 
    goto choice

    :A 
    @echo off 
    netsh interface ip set address name = "Ethernet" source = static addr = 192.168.1.99 mask = 255.255.255.0 gateway = 192.168.1.1
    netsh interface ipv4 add dnsserver "Ethernet" address=192.168.1.1 index=1
    goto end

    :B 
    @ECHO OFF
    netsh int ip set address name = "Ethernet" source = dhcp
    netsh int ip set dns name = "Ethernet" source = dhcp

    ipconfig /renew Ethernet

    goto end 
    :end

When you run the batch file you will be prompted A:VPN or B:Local ISP

It all seems hard but its easier then you think :)

Enjoy!



 





        
      



    