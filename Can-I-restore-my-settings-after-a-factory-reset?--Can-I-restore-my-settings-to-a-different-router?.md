## Description #
There are times when it is necessary to perform a factory reset, such as when loading firmware where the wireless drivers have been updated.  The Save/Restore configuration options in the firmware **cannot** be used to restore your user settings in this case since there is driver specific information also included in the save confirguration.  Therefore the user has been required to reconfigure the personalized settings through the GUI by hand after the update.  Additionally, if incorporating a new router to replace an existing router, even of the same model, the firmware Save/Restore options cannot be used and a fresh setup would be required.

A simple script/utility is available that reads from a configuration (ini) file the nvram variables associated with parts of the user configuration (excluding those unique to the router) and creates a restore script for those variables. This replaces the process of re-entering all the customized settings through the web GUI. 

A 'Migration Mode' has also been added for use when transferring settings between routers. This mode excludes certain variables (such as transmit power), that should evaluated first on the new hardware.

## Obtaining the Utility #
The latest version of the utility can be found in the Asuswrt-Merlin subforum on SmallNetBuilders Forum.  Included in the download zipfile is a QuickStart.txt that goes step-by-step on how to use a USB stick with the utility.
 
[User NVRAM Save/Restore Utility](http://forums.smallnetbuilder.com/showthread.php?t=19521)