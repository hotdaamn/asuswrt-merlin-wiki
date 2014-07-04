## Introduction ##

Welcome! This guide will show you how to compile Asuswrt-Merlin from source, with the final result being built firmware images that can be flashed to your router.

> **HERE BE DRAGONS!** There are risks when compiling your own firmware. You do so at your own risk, and your compiled firmware cannot be supported by the firmware author should something go wrong. You should be aware and familiar with the Asus recovery utilities and how to access and flash firmware from recovery mode. Your firmware may not build, it may not flash, it may flash but cause the router but not boot, it may flash and boot but work incorrectly or in unexpected ways, your computer may spontaneously combust, you may spontaneously combust, or the world may end.

## Prerequisites ##

**Note: this guide assumes a basic working knowledge of a Linux distribution. You should know how to install and access your Linux distribution, access the terminal, and install packages from your distribution's package manager.**

**Note: this guide does not cover downloading the Asuswrt-Merlin source for compilation - refer to [this guide](https://github.com/RMerl/asuswrt-merlin/wiki/Download-the-latest-source-code-from-GitHub) to acquire the source code.**

**Note: this guide is targeted towards Debian-based distributions, such as Ubuntu and Linux Mint. This guide can still be used as reference on non-Debian distributions, but you may need to substitute other commands (most notably the `apt-get` commands) with commands suitable for your system's package manager. You may also require different packages to be installed on your system should you be running a 64-bit build environment. Refer to your distribution's documentation for more information.**

**Note: if you are using a virtual machine, the compilation process may take a long time. You may want to ensure hardware virtualization is enabled in your system's BIOS (if applicable) and that your virtual enviroment is configured to use hardware virtualization and more than one processor core. Refer to the virtual machine's documentation for details on how to configure these settings.**

To compile a firmware image, you will need the following:

* A Linux-based operating environment (either a dedicated machine running Linux, or a virtual machine using [VMware](http://www.vmware.com) or [VirtualBox](https://www.virtualbox.org/)).
* At least 1GB free drive space

## Preparing the Build Environment ##

Before we can build Asuswrt-Merlin, we need to install packages required that will be used during the build process to compile the firmware. Open a terminal window and type or paste in the following:

    sudo apt-get install autoconf automake bash bison bzip2 diffutils file flex g++ gawk gcc-multilib gettext gperf groff-base libncurses-dev libexpat1-dev libslang2 libssl-dev libtool libxml-parser-perl make patch perl pkg-config python sed shtool tar texinfo unzip zlib1g zlib1g-dev

When prompted, enter your password for `sudo` to gain root access to install the packages. Enter 'Y' when prompted to install the packages. When the packages have finished installing, you will be returned back to your command line.

**Note: the following commands assume that you have downloaded the source code into `~/asuswrt-merlin/` as specified in the [Download the latest source code from GitHub](https://github.com/RMerl/asuswrt-merlin/wiki/Download-the-latest-source-code-from-GitHub) guide on this wiki. If this is not where you have downloaded the source code to, adjust the commands as necessary for your own environment.**

Now we must append to our system's PATH environment variable so the build system can find the toolchain required for cross-compilation. First start by creating a symbolic link to make appending easier:

    sudo ln -s ~/asuswrt-merlin/tools/brcm /opt/brcm

Now update our PATH environment variable (note that you will need to rerun this command each time you open your terminal in future compilations):

    export PATH=$PATH:/opt/brcm/hndtools-mipsel-linux/bin:/opt/brcm/hndtools-mipsel-uclibc/bin

## Building the Firmware ##

We are now ready to compile the firmware. Keep in mind that this process will take some time, so feel free to grab a coffee or go for a walk while the firmware builds.

**If you are building for the Asus RT-N66U:**

Change into the build directory:

    cd ~/asuswrt-merlin/release/src-rt-6.x/

Perform the build:

    make rt-n66u

When the build completes, the newly created firmware image will be located in `~/asuswrt-merlin/release/src-rt/image/`.

**If you are building for the Asus RT-AC66U:**

Change into the build directory:

    cd ~/asuswrt-merlin/release/src-rt-6.x/

Perform the build:

    make rt-ac66u

When the build completes, the newly created firmware image will be located in `~/asuswrt-merlin/release/src-rt-6.x/image/`.

## Cleaning up ##

As code changes and becomes updated, you may wish to recompile your firmware images to use more current versions of the source code. It's a good idea to clean your build environment to remove leftover files from previous builds and ensure that they won't cause conflicts during a new build.

**For the Asus RT-N66U:**

Change into the build directory:

    cd ~/asuswrt-merlin/release/src-rt-6.x/

Perform the build clean:

    make clean

**For the Asus RT-AC66U:**

Change into the build directory:

    cd ~/asuswrt-merlin/release/src-rt-6.x/

Perform the build clean:

    make clean

## Conclusion ##

That's it! This guide should have helped you in compiling the Asuswrt-Merlin source code into firmware images suitable for flashing onto your router.