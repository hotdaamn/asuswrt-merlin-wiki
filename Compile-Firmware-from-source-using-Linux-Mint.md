# How to build `asuswrt-merlin` on Linux Mint

## Cautionary note

> WARNING DESPITE THE FACT THAT IT IS POSSIBLE TO COMPILE YOUR OWN FIRMWARE,
> I DO NOT ADVISE YOU TO DO SO
> THIS GUIDE IS NOT WRITTEN BY A PROFESSIONAL BUT BY AN AMATEUR<>NOVICE
> THERE ARE A LOT OF THINGS THAT COULD GO WRONG DURING THE PROCESS
> THE AUTHOR OF THIS GUIDE NOR THE SOURCE PROVIDER CAN BE HELD RESPONSIBLE FOR ANY DAMAGE THAT MAY OCCUR BY 
> FLASHING YOUR SELF COMPILED FIRMWARE ON YOUR ROUTER
> IF YOU REALLY WANT TO LEARN THIS TYPE OF COMPUTING AND CODING I SUGGEST YOU FIRST GET FAMILIAR WITH LINUX

But on the other hand your router is VERY hard to brick!

## Additional information

Linux Mint uses the Ubuntu repositories for a large portion of its packages. Or differently put: Linux Mint is a derivative of Ubuntu. You should be able to apply [the instructions for Ubuntu](https://github.com/RMerl/asuswrt-merlin/wiki/Compile-Firmware-from-source-using-Ubuntu) also on Linux Mint.

Make sure to check out the Wikipedia article on the [Linux Mint releases](http://en.wikipedia.org/wiki/List_of_Linux_Mint_releases), to see what Ubuntu release corresponds to your specific Linux Mint release.

If you have Linux Mint installed already, use `lsb_release -a` from a terminal to find out the version you are running.

## Example: building under Linux Mint 13 (Maya)

For the reference we are going to use Linux Mint 13 in Virtualbox.

* You can download Linux Mint 13 [here](http://www.linuxmint.com/download.php)
* And Virtualbox [here](https://www.virtualbox.org/wiki/Downloads)

When you have them both install VirtualBox

Then follow these steps :

* Create a new virtual machine 

![Create](http://members.home.nl/frits.pruymboom/Compile%20with%20linux%20mint/1.png)


* After you created your new VM its a good case to give it some horsepower because the compiling will go a little faster, if you have enough time you can skip this step.

![crank](http://members.home.nl/frits.pruymboom/Compile%20with%20linux%20mint/Crank.png)

* Fire up MINT 13 and install everything.
* When MINT 13 is installed, fire up a terminal by pressing CTRL+ALT+T

![terminal](http://members.home.nl/frits.pruymboom/Compile%20with%20linux%20mint/terminal.png)

## Manually preparing the build environment

From here on it is just copy and paste work !

* Just paste in the commands because some commands require ROOT privilege we use `sudo`.
* It will ask you for a password type the password you provided during install.  

> **Note:** the `root` account is actually only required for certain parts of the whole procedure, specifically for the symlinking of the toolchain into `/opt`. If you fix up the hardcoded paths in some of the source files, you will be able to run the whole build procedure without superuser privileges. If you prefer to automate the following steps, scroll down to the section named *Automated all-in-one script*.

* Install the base packages you need to compile

```bash
sudo apt-get install bison flex g++ g++-4.4 g++-multilib gawk gcc-multilib gconf-editor gitk lib32z1-dev libncurses5 libncurses5-dev libstdc++6-4.4-dev libtool m4 pkg-config zlib1g-dev gperf lib32z1-dev libelf1:i386
libmpc2:i386
```

* Download Merlins hard work

**Note:** a very detailed guide on how to download the source code can be found [on this Wiki page](https://github.com/RMerl/asuswrt-merlin/wiki/Download-the-latest-source-code-from-GitHub).

```bash
git clone https://github.com/RMerl/asuswrt-merlin.git
```

* Let Linux know where to search for the tools to compile

```bash
sudo ln -s $HOME/asuswrt-merlin/tools/brcm /opt/brcm
export PATH=$PATH:/opt/brcm/hndtools-mipsel-linux/bin:/opt/brcm/hndtools-mipsel-uclibc/bin
sudo mkdir -p /media/ASUSWRT/
sudo ln -s $HOME/asuswrt-merlin/media/ASUSWRT/asuswrt
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

## Notes for Linux Mint 16 (Petra) and later

This has been taken verbatim from the Ubuntu instructions as it should also apply here.

> If you want to build in Ubuntu 13.10, before you `make clean` / `make [router]`, you might need to perform these steps due to the different version of `autoconf`.

> This works to build 3.0.0.4_374.38_1.

> ```bash
> sudo apt-get install libproxy-dev
> # fix neon missing proxy.h
> cp /usr/include/proxy.h ~/asuswrt-merlin/release/src/router/neon/
> # fix broken configure script for libdaemon
> cd ~/asuswrt-merlin/release/src/router/libdaemon
> aclocal
> # fix broken configure script for libxml2
> cd ~/asuswrt-merlin/release/src/router/libxml2
> sed -i s/AM_C_PROTOTYPES/#AM_C_PROTOTYPES/g ~/asuswrt-merlin/release/src/router/libxml2/configure.in
> aclocal
> ```

## Automated all-in-one script

**Status:** This has been tested on Ubuntu 12.04 and 14.04 and is known to work as of 2015-02-02. That means it has **not** been tested on Linux Mint specifically. Please report back about issues and successes.

The latest version can always be found at [assarbad/build-asuswrt-merlin](https://github.com/assarbad/build-asuswrt-merlin). Please report issues there as well.

On the landing page (above link) you will find a brief introduction on how to use the script as well as its status.