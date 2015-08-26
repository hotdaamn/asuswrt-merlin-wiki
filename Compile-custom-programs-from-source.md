I had need for an upgraded Arpwatch that supported VLAN tags to fix the 

```text
arpwatch: 0:50:4:fe:7e:3c sent bad hardware format 0x800
```

errors that appeared in my logs, its fairly easy to compile once you have the toolchain properly installed and this is how to do it:

First follow one of these links to setup your toolchain, ignore the very last step which compiles the firmware.

* [Compiling from source using a Debian based Linux Distribution](/RMerl/asuswrt-merlin/wiki/Compiling-from-source-using-a-Debian-based-Linux-Distribution)
* [Compile Firmware from source using Linux Mint](/RMerl/asuswrt-merlin/wiki/Compile-Firmware-from-source-using-Linux-Mint)
* [Compile firmware from source using Ubuntu](/RMerl/asuswrt-merlin/wiki/Compile-Firmware-from-source-using-Ubuntu)

Instead of running `make DEVICEMODEL`, run:

```bash
 make clean 2>&1 | grep -oE '(mipsel|arm).*gcc' | uniq
```

Then change into the directory of the program you want to compile and run:

```bash
./configure CC=OUTPUTFROMGREP --prefix=/opt --host=arm-linux
make
```

Then you can simply copy the executable over to your jffs or usb drive.