### General question about Asuswrt-Merlin

**Q: Why this firmware?**  
A: At first, because I wanted something just a little bit better than the original firmware.  Then I wanted something more stable.  And finally it was a fun thing to do and there was enough interest out there for me to turn this into a full project.

**Q: Why not add feature "X"?  It would be useful.**  
A: The first reason why a feature might be rejected is because I try to change as little as possible to the original firmware.  This ensures a limited chance of introducing new bugs.  This ensures that whenever Asus releases a new version, it won't take too much time for me to rebase all my work on the new version they released.  And also because there are already excellent feature-packed alternatives out there with Tomato and DD-WRT.  The goal of Asuswrt-Merlin is not to reproduce these two projects, but to enhance on the original firmware.  If you need more features and these can't be added through Optware, then you should probably look at Tomato or DD-WRT.

And the second reason is since I'm alone working on this and I'm doing it on my spare time, I have limited time to devote to it, and have to assign priorities.  And the current chain of priorities goes like this:

_Stability > performance > features_


**Q: Does Asus know about this project?**  
A: Yes.  We frequently exchange emails, and they have also integrated some of the features and bug fixes from this firmware project into the stock firmware.  So far they have been quite supportive of this project, for which I am very grateful.

**Q: Will flashing this void my warranty?**  
A: This is a kinda grey area for which I have no definitive answer unfortunately.  I have seen Asus publicly commit to supporting DD-WRT for some of their models, without any mention about it voiding your warranty.  Therefore my personal opinion would be that "No, it won't void your warranty", but as the saying goes, IANAL.  Your retailer might certainly have a different view on this than Asus would, too.

**Q: Will you support router "RT-xXX" from Asus?**  
A: Most likely not.  Without having an actual router to test with, building a firmware that people would flash without me having at least confirmed it can be flashed at all would be far too risky.


### Recovery and resetting

**Q: How do I put the router into Recovery Mode?**  
A: Turn the router off.  Press the reset button, and while keeping it pressed turn it back on.  Wait a few seconds until the power led blinks, and then release the reset button.  Router will now be in Recovery Mode, reachable at 192.168.1.1.

**Q: How do I wipe my settings?**  
A: Press the reset button for more than 5 seconds, then release it.

**Q: How do I wipe my settings?  Reset button does not work.**  
A: If the firmware fails to boot, then the reset button won't work.  Turn the router off, press the WPS button.  While keeping it pressed turn the router back on.  Wait a few seconds, then release the WPS button.  Settings will be back to factory defaults.

**Q: Is there a risk of bricking my router with this firmware?**   
A: Virtually none.  The firmware update does not touch the CFE/Bootloader, so you will always be able to enter Recovery Mode to reflash any firmware if anything went wrong with flashing.
