This script is intended for people that want to wake up their server when someone tries to access a port on the server.
The script doesn't mind which port was used and will respond to any request so if you have much forwarded ports a port scan will wake up your web server every time.

I rewrote the script a little to fit my own needs
On the bottom of this page you will find another script, that script is intended for people that only want to wake a server on one port.


Guide For Wake On Lan Script :

- Go to your router and login (192.168.1.1) With username and password
- Go to administration and enable JFFS partition + Format JFFS partition at next boot
- Enable SSH And Telnet

![foto1](http://members.home.nl/frits.pruymboom/Enable%20JFFS%20&%20Telnet%20SSH%201.PNG)
- Hit Apply and reboot your router !
- Download WINSCP [Download](http://winscp.net/download/winscp439setup.exe"]http://winscp.net/download/winscp439setup.exe)
- Start WINSCP and login
![foto 2](http://members.home.nl/frits.pruymboom/Login%20WINSCP%202.PNG)

- Be sure to select protocol SCP, your username and password are the ones you setup for your router.

( WINSCP will give a warning about some groups missing, this is fine doesnt affect your job )

- Go to your root folder and click JFFS
![Foto3](http://members.home.nl/frits.pruymboom/Root%20folder%20WINSCP%203.PNG)

- There should be a folder scripts ( if not there create it ! )

- Inside the folder JFFS right click >New>File. Name the file services-start
Paste in the content.

```
#!/bin/sh

#script for sending WOL packets when traffic to specified ip's
sh /jffs/scripts/wake.sh& 
```

![foto4](http://members.home.nl/frits.pruymboom/SCRIPT%20SERVICES%20START%205.PNG)

- Create another file name it wake.sh and paste in
```
#!/bin/sh

INTERVAL=5
NUMP=1
OLD=""
TARGET=IP OF THE TARGET YOU WANT TO WAKE
IFACE=br0
MAC=MAC ADRESS OF THE TARGET YOU WANT TO WAKE
WOL=/usr/bin/ether-wake
LOGFILE="/var/log/ether-wake.log"

while sleep $INTERVAL;do
NEW=`dmesg | awk '/ACCEPT/ && /DST='"$TARGET"'/ {print }' | tail -1`
SRC=`echo $NEW | awk -F'[=| ]' '{print $8}'`
DPORT=`echo $NEW | awk -F'[=| ]' '{print $27}'`
PROTO=`echo $NEW | awk -F'[=| ]' '{print $23}'`

if [ "$NEW" != "" -a "$NEW" != "$OLD" ]; then
if ! ping -qc $NUMP $TARGET >/dev/null; then
# echo "NOWAKE $TARGET was accessed by $SRC, port $DPORT, protocol $PROTO and is already alive at" `date`>> $LOGFILE
# else
echo "WAKE $TARGET requested by $SRC, port $DPORT, protocol $PROTO at" `date`>> $LOGFILE
$WOL -i $IFACE $MAC
sleep 5
fi
OLD=$NEW
fi
done  
```


- Put the IP and MAC in the script

![foto5](http://members.home.nl/frits.pruymboom/SCRIPT%20WAKE%20SH%204.PNG)

Now it should look like this
![Foto6](http://members.home.nl/frits.pruymboom/Win%20scp%20round%20up%206.PNG)

Now we need to do some telnet ssh command.

Download PUTTY [Download](http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe"]http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe)

- Go ahead and login with putty ( 192.168.1.1 )

![foto9](http://members.home.nl/frits.pruymboom/Putty%20login%207.PNG)

- Give the router username and password

Now copy and paste into putty


```
chmod +x /jffs/scripts/wake.sh

chmod +x /jffs/scripts/services-start
```



With those command you made your files executable.

We need to do one more final step !

- Login to your router webpage 192.168.1.1 > go to firewall and set logged packets to ACCEPTED
- Hit apply
Now reboot your router , Its time for some testing

![foto10](http://members.home.nl/frits.pruymboom/Firewall%20packets%207.PNG)



**ALTERNATIVE**

Use this script if you only want to specify one port to wake up your server

```
#!/bin/sh

INTERVAL=5
NUMP=3
OLD=""
TARGET=PUT YOUR INTERNAL IP ADRESS HERE!
PORT=SPECIFY YOUR PORT HERE!
IFACE=br0
MAC=PUT YOUR MAC ADRESS HERE!
WOL=/usr/bin/ether-wake
LOGFILE="/var/log/ether-wake.log"

while sleep $INTERVAL;do
NEW=`dmesg | awk '/ACCEPT/ && /DST='"$TARGET"'/ && /DPT='"$PORT"'/ {print }' | tail -1`
SRC=`echo $NEW | awk -F'[=| ]' '{print $8}'`
DPORT=`echo $NEW | awk -F'[=| ]' '{print $27}'`
PROTO=`echo $NEW | awk -F'[=| ]' '{print $23}'`

if [ "$NEW" != "" -a "$NEW" != "$OLD" ]; then
if ! ping -qc $NUMP $TARGET >/dev/null; then
# echo "NOWAKE $TARGET was accessed by $SRC, port $DPORT, protocol $PROTO and is already alive at" `date`>> $LOGFILE
# else
echo "WAKE $TARGET requested by $SRC, port $DPORT, protocol $PROTO at" `date`>> $LOGFILE
$WOL -i $IFACE $MAC
sleep 5
fi
OLD=$NEW
fi
done 
```
