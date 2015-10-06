* Work in progress
* If possible define verification (test) criteria for each step

[Source: SNB forums](http://www.snbforums.com/threads/pixelserv-a-better-one-pixel-webserver-for-adblock.26114/)

### ARM Pixelserv Installation Process
1. [Copy Arm Pixelserv binary to /opt/sbin:] (http://www.linksysinfo.org/index.php?threads/pixelserv-compiled-to-run-on-router-wrt54g.30509/page-9#post-263530)

2. [Step 1: Install PixelServer]
`ENABLED=yes
PROCS=pixelserv
ARGS="192.168.1.1 -p 8080"
PREARGS=""
DESC=$PROCS
PATH=/opt/sbin:/opt/bin:/opt/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

. /opt/etc/init.d/rc.func`



3. [Step 2: Download Blacks list]
4. [Step 3: ????]