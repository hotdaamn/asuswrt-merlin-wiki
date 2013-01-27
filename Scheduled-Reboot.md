The following user script will make your router reboot itself every night at 4 am:

```
#!/bin/sh
cru a ScheduledReboot "0 4 * * * reboot"
```

Put this inside an init-start user script.
