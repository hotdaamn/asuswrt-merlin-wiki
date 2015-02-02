# How to build `asuswrt-merlin` on Ubuntu

## Cautionary note

> WARNING DESPITE THE FACT THAT IT IS POSSIBLE TO COMPILE YOUR OWN FIRMWARE,
> I DO NOT ADVISE YOU TO DO SO
> THIS GUIDE IS NOT WRITTEN BY A PROFESSIONAL BUT BY AN AMATEUR<>NOVICE
> THERE ARE A LOT OF THINGS THAT COULD GO WRONG DURING THE PROCESS
> THE AUTHOR OF THIS GUIDE NOR THE SOURCE PROVIDER CAN BE HELD RESPONSIBLE FOR ANY DAMAGE THAT MAY OCCUR BY 
> FLASHING YOUR SELF COMPILED FIRMWARE ON YOUR ROUTER
> IF YOU REALLY WANT TO LEARN THIS TYPE OF COMPUTING AND CODING I SUGGEST YOU FIRST GET FAMILIAR WITH LINUX

But on the other hand your router is VERY hard to brick!

## Example: building under Ubuntu 12.04

For the reference we are going to use Ubuntu 12.04 in VMware Player.

* You can download Ubuntu [here](http://www.ubuntu.com/download)
* And VMware Player [here](https://my.vmware.com/web/vmware/free#desktop_end_user_computing/vmware_player/5_0)

When you have both install VMware Player.

Follow these steps :

* Create a New Virtual Machine

![Create](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/Create%20new%20virtual%20mashine.png)

* Select Installer disc image file (iso) ( VMware Player will tell you something about easy install, that's okay )

![Select Iso]
(http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/installer%20disc%20iso.png)

* When installing Ubuntu it asks for a username it's best if you just name it ''router'' without the quotes because then you can just copy and paste from this guide no need to make everything complicated.

![Name](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/important%20name.png)

* Give it about 20 GB afterwards you can delete your virtual Ubuntu anyway.

![20gb](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/20gb.png)

* Crank up some specs give it some more ram and procesing power, It doesn't sound as a heavy work load but making your .trx image for the router actually takes a long time on a slow processor

![customize](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/customiza%20hardware.png)

![crank](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/crank%20up%20the%20specs.png)

When the Ubuntu installer has finished you can fire up terminal

* Fire up a terminal CTRL+ALT+T

## Manually preparing the build environment

We are going to make the root account active because if we don't do that it will give us a lot of errors during compilation.

> **Note:** the `root` account is actually only required for certain parts of the whole procedure, specifically for the symlinking of the toolchain into `/opt`. If you fix up the hardcoded paths in some of the source files, you will be able to run the whole build procedure without superuser privileges. If you prefer to automate the following steps, scroll down to the section named *Automated all-in-one script*.

Paste in the following lines in terminal

```bash
sudo passwd root 
```

* It will now ask you for your new `root` password (confirm this twice and be sure to remember it we need it later on!)

Paste in

```bash
su
```

As you can see you are now running the terminal in root!

![root](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/Root.png)

* We are going to download some packages ( I am sure there are some you don't need, but this is how I got it working and you can delete Ubuntu afterward so who cares right ? )

Paste in:

```bash
apt-get install autoconf automake bash bison bzip2 diffutils file flex m4 \
g++ gawk groff-base libncurses-dev libtool libslang2 make patch perl pkg-config \
shtool subversion tar texinfo zlib1g zlib1g-dev git-core gettext libexpat1-dev \
libssl-dev cvs gperf unzip python libxml-parser-perl gcc-multilib gconf-editor \
libxml2-dev g++-4.4 g++-multilib gitk libncurses5 mtd-utils libncurses5-dev \
libstdc++6-4.4-dev libvorbis-dev g++-4.4-multilib git autopoint autogen sed \
build-essential intltool libelf1:i386 libglib2.0-dev automake-1.11
```
Assuming you don't run as `root`, make sure to prepend a `sudo` to the above command line.

If you have Ubuntu x64 edition then you need `lib32z1-dev` and `lib32stdc++6`

```bash
sudo apt-get install lib32z1-dev lib32stdc++6
```
If you are ready installing all these packages take a coffee or a beer.

We are going to download Merlins hard work with again some terminal commands.  It appears as if scripts published in support forums all assume the clone is done off of root, so first we change to the root directory.

**Note:** a very detailed guide on how to download the source code can be found [on this Wiki page](https://github.com/RMerl/asuswrt-merlin/wiki/Download-the-latest-source-code-from-GitHub).

```bash
cd /root
git clone https://github.com/RMerl/asuswrt-merlin.git
```

Be patient it takes some time

With these command you will build your environment which you will need to work with.

```bash
sudo ln -s ~/asuswrt-merlin/tools/brcm /opt/brcm
sudo ln -s ~/asuswrt-merlin/release/src-rt-6.x.4708/toolchains/hndtools-arm-linux-2.6.36-uclibc-4.5.3 /opt/brcm-arm
```

Adjust the `PATH` environment variable:

```bash
export PATH=$PATH:/opt/brcm/hndtools-mipsel-linux/bin:/opt/brcm/hndtools-mipsel-uclibc/bin:/opt/brcm-arm/bin
```

Set one more symlink:

```bash
sudo mkdir -p /media/ASUSWRT/
```

```bash
sudo ln -s ~/asuswrt-merlin /media/ASUSWRT/asuswrt-merlin
```

## Ready to build: manual approach

### RT-N16

```
cd ~/asuswrt-merlin/release/src-rt
make clean
make rt-n16
```

### RT-N66U & RT-AC66U

```
cd ~/asuswrt-merlin/release/src-rt-6.x
make clean
```

#### ... then for RT-N66U

```
make rt-n66u
```

#### ... and for RT-AC66U

```bash
make rt-ac66u
```

### RT-AC56U & RT-AC68U

```bash
cd ~/asuswrt-merlin/release/src-rt-6.x.4708
make clean
```

#### ... then for RT-AC56U

```bash
make rt-ac56u
```
#### ... and for RT-AC68U

```bash
make rt-ac68u
```

## Notes for Ubuntu 13.10 and later

If you want to build in Ubuntu 13.10, before you `make clean` / `make [router]`, you might need to perform these steps due to the different version of `autoconf`.

This works to build 3.0.0.4_374.38_1.

```bash
sudo apt-get install libproxy-dev
# fix neon missing proxy.h
cp /usr/include/proxy.h ~/asuswrt-merlin/release/src/router/neon/
# fix broken configure script for libdaemon
cd ~/asuswrt-merlin/release/src/router/libdaemon
aclocal
# fix broken configure script for libxml2
cd ~/asuswrt-merlin/release/src/router/libxml2
sed -i s/AM_C_PROTOTYPES/#AM_C_PROTOTYPES/g ~/asuswrt-merlin/release/src/router/libxml2/configure.in
aclocal
```

## Automated all-in-one script

**Status:** This has been tested on Ubuntu 12.04 and 14.04 and is known to work as of 2015-02-02.

The latest version can always be found at [assarbad/build-asuswrt-merlin](https://github.com/assarbad/build-asuswrt-merlin). Please report issues there as well.

On the landing page (above link) you will find a brief introduction on how to use the script as well as its status.