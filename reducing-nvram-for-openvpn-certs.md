
### Reducing nvram usage because of large certificates/date
If you are like me, I run 2 vpn servers and 2 vpn clients at any time.  I find that I always run very close to the edge of running out of nvram.  What I do now is I use my jffs volume to hold the certificates rather than storing them in nvram.  

I figured i'd share a quick 'how to' in case you wanted to do this.
1) Copy all your certificates out so you have them.
2) Create a folder structure to your liking on /jffs.  THis is mine:

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

4) Remove the nvram entries.  You can do this either by command line.  or you can just put a single space in each field holding the certificates you are moving to nvram and apply in the GUI.

5) add entries like this to the advanced config page.
     `tls-auth /jffs/ov/s/static.key 0`
     ca /jffs/ov/s/ca.crt
     dh /jffs/ov/s/dh.pem
     cert /jffs/ov/s/server.crt
     key /jffs/ov/s/server.key

You can consult this page for details of the actual parameters.
https://openvpn.net/index.php/open-source/documentation/howto.html#server
