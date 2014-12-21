I had need for an upgraded Arpwatch that supported VLAN tags to fix the 

arpwatch: 0:50:4:fe:7e:3c sent bad hardware format 0x800

errors that appeared in my logs, its fairly easy to compile once you have the toolchain properly installed and this is how to do it:

First follow this link to setup your toolchain, ignore the very last step which compiles the firmware.

[Compile-Firmware-from-source-using-MINT-13](/RMerl/asuswrt-merlin/wiki/Compile-Firmware-from-source-using-MINT-13)

Instead of running "make DEVICEMODEL", run :

"make clean 2>&1 | grep -o 'arm.*gcc'"

Than change into the directory of the program you want to compile and run:

./configure CC=OUTPUTFROMGREP --prefix=/opt --host=arm-linux

make

Then you can simply copy the executable over to your jffs or usb drive.