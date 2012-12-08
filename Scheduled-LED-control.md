Starting with 3.0.0.4.246.20, the router LEDs can now be controlled by the user.  The simplest way is to enable the Stealth Mode option (under Tools -> Other Settings), which will disable them.  It is also possible to turn them on and off on a specific schedule through cron jobs.

The following example will let you configure your router so that LEDs will turn themselves off at 18:00, and turn back on at 6:00.

First, make sure you have enabled the [JFFS](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS) partition.  Next, either create the following services-start script (or add to any existing script):

services-start:
```
#!/bin/sh
cru a lightsoff "0 18 * * * /jffs/scripts/ledsoff.sh"
cru a lightson "0 6 * * * /jffs/scripts/ledson.sh"
```
You can adjust the time at which you want both events to fire off by adjusting the second digit (which represents the hour).

Then, create the following two scripts, and also save them in /jffs/scripts/ :

ledsoff.sh:
```
#!/bin/sh
nvram set led_disable=1
nvram commit
service restart_leds
```

ledson.sh:
```
#!/bin/sh
nvram set led_disable=0
nvram commit
service restart_leds
```

Then, reboot your router.  You can confirm that the jobs are properly set up by running the following command through telnet:

```
cru l
```

You should see both scheduled tasks listed.
