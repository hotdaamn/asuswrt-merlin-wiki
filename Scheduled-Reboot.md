The following user script will make your router reboot itself every night at 4 am:

```
#!/bin/sh
cru a ScheduledReboot "0 4 * * * /sbin/reboot"
```

Put this inside an `/jffs/scripts/init-start` [user script](https://github.com/RMerl/asuswrt-merlin/wiki/User-scripts).
