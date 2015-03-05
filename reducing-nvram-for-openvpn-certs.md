
### Reducing nvram usage because of large certificates/date
If you are like me, I run 2 vpn servers and 2 vpn clients at any time.  I find that I always run very close to the edge of running out of nvram.  What I do now is I use my jffs volume to hold the certificates rather than storing them in nvram.  

I figured i'd share a quick 'how to' in case you wanted to do this.

1) Copy all your certificates out so you have them.

** Note: If you want an easy way to export the certificates you currently have in nvram to files, you can use something like this.  This is just quick and dirty.  It will dump the nvram paramters to your current directory based on the name of hte entry.  Then you can move them around as you see fit.

    mkdir /jffs/ov
    nvram show | egrep ^vpn_crt | cut -d"=" -f1 | while read e; do   
     nvram get $e > /jffs/ov/$e
    done 

2) Create a folder structure to your liking on /jffs.  This is mine:

     /jffs/ov
     /jffs/ov/s  <- server ones.  My 2 servers use the same certs but you could also make this 's1' and 's2'
     /jffs/ov/c  <- client ones.  My 2 clients use the same certs but you could also make this 'c1' and 'c2'

3) Create your certificates in your structure (example):

    /jffs/ov/c/ca.crt
    /jffs/ov/s/ca.crt
    /jffs/ov/s/ca.key
    /jffs/ov/s/dh.pem
    /jffs/ov/s/server.crt
    /jffs/ov/s/server.key
    /jffs/ov/s/static.key

vram entries and bypass the GUI restriction requiring them to have something in them.  To do this, you need to put a single space in each field on the "content modification of Keys & Certification" link in the GUI. This is so the GUI thinks you have values in there...  But the real content is overriden by the entries and the jffs data.

5) add entries similar to these to the advanced settings -> custom configs page in the GUI depending on what certs you use and what path you put them in.  For tls-auth, you need a zero or a one after the name depending on if it is a client (1) or server (0) entry.  Consult the openvpn link below.

     tls-auth /jffs/ov/s/static.key 0
     ca /jffs/ov/s/ca.crt
     dh /jffs/ov/s/dh.pem
     cert /jffs/ov/s/server.crt
     key /jffs/ov/s/server.key

You can consult this page for details of the actual parameters.
https://openvpn.net/index.php/open-source/documentation/howto.html#server
