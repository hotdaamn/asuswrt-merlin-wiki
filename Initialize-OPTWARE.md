This is a simple tutorial for initializing optware on your Asus router using Asuswrt-Merlin.
After a long time search I found out that the best way to initialize optware on your router is to install Download Manager.

Afterwards you can always delete Download Manager.

I am going to guide you trough the basics, don't expect too much from this tutorial.
At this moment there are not a lot of projects that are specifically for Asuswrt-Merlin, so if you want to install something on optware you would really need to adjust a lot of parameters.

But cut to the point

First of all you need a USB stick or you should install a mircoSD card in the router.
For the guide I will use a USB stick because if you screw things up the microSD is ''inside'' the router so it will be very difficult to get it out.
But with a USB stick.......... If you screw it up then you can just unplug the USB and reboot your router.

This is the easy way!

You need to format your USB Stick to EXT 2 on a Linux machine this isn't the problem you can just use gparted or something.
For Windows it's a bit difficult to find suitable software but this is what I found on Google and actually it works quite well and it is **FREE**

[MiniTool Partition Wizard Home Edition](http://www.partitionwizard.com/free-partition-manager.html)

Now let's show you how to format your USB stick into EXT2

* Start the program
* Find your USB Stick

![1](http://members.home.nl/frits.pruymboom/Initialize%20optware%20foto1.png)

* Right click on it and delete any older partitions or leftovers ( YOU WILL LOSE ALL YOUR DATA )
* Now right click on it again and select **Create**
* We are going to make a 2 GB partition for optware, that's really what we need optware doesn't use much space so but Download Master cant do with it.
* Of course if you have a big USB stick you can make some other partitions on it too !

![2](http://members.home.nl/frits.pruymboom/Initialize%20optware%20foto2.png)

* Hit OK
* Hit apply

When that is done plug it in one of the USB slots of your router


**NOW INSTALL DOWNLOAD MASTER**

*Go to ( 192.168.1.1 ) Login with your username and password

**FOLLOW THE PICTURE**

![install download master](http://members.home.nl/frits.pruymboom/Install%20download%20master%201.png)


**Once successfully installed we are going to remove it. Not disable it but really uninstall**

![uninstall](http://members.home.nl/frits.pruymboom/Deinstal%20download%20master%201.png)

Now i know you were wondering why install something if we are going to delete it anyway,
Well that's the job that asus did for us so we don't have to mount optware because if you would do it with command line there is a lot that you can mount at the wrong point.
And the leftovers from Download Master are the basic maps that you need to run optware.