**Q : Does Asus-Merlin WRT auto detects CFE if i flashed my CFE to 64K ?**

A : Yes it Does


**Q : Is a CFE update to 64K really necessary for the RT-N66U ?**

A : It is recommended you just stay with your original asus CFE 32K in this case, Asus-Merlin WRT gives you 64K at kernel level without risking your router flashing your CFE stays a risky business.

**Q : Does UPnP have a port number limit**

A : Yes it does not allow you to set a port lower than 1024, there is a solution to program directly into NVRAM 

Example:

```
nvram set upnp_min_port_int=1
nvram set upnp_min_port_ext=1
nvram commit
```

**Q : Is the log-out button faulty ?**

A : No, This is a known problem and it is not caused by the routers firmware but rather by your browser it caches the HTTP password

**Q : Does Asus-Merlin WRT auto update ?**

A : No that function is removed in so you can't flash the original firmware by accident. 

**Q : Is the included lighttpd full featured ?**

A : Yes it is but the downside is AIcloud uses some proprietary modules, so most likely you will run into some troubles. 
But there are some alternatives in the optware repo

**Q : Can i run optware from the JFFS partition ?**

A : It is not recommended to do so, JFFS is flash storage and some optware services do a lot of writes.
That coul prematurely wear out the flash chips inside the router, It would be better to install a micro-SD or to use a USB stick or HDD. 

