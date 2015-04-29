# How to build `asuswrt-merlin` on Debian-based distributions

## Introduction

Welcome! This guide will show you how to compile `asuswrt-merlin` from source, with the final result being built firmware images that can be flashed to your router.

> **HERE BE DRAGONS!** There are risks when compiling your own firmware. You do so at your own risk, and your compiled firmware cannot be supported by the firmware author should something go wrong. You should be aware and familiar with the Asus recovery utilities and how to access and flash firmware from recovery mode. Your firmware may not build, it may not flash, it may flash but cause the router but not boot, it may flash and boot but work incorrectly or in unexpected ways, your computer may spontaneously combust, you may spontaneously combust, or the world may end.

## Scope

This guide is targeted towards Debian-based distributions, such as:

* **Debian** itself
* **Ubuntu**
* **Linux Mint**

This guide can still be used as reference on non-Debian distributions, but you may need to substitute other commands (most notably the `apt-get` commands) with commands suitable for your system's package manager (e.g. `yum`). You may also require different packages to be installed on your system should you be running a 64-bit build environment. Refer to your distribution's documentation for more information.

**Note:** this guide does not cover downloading the `asuswrt-merlin` source for compilation - refer to [this guide](/RMerl/asuswrt-merlin/wiki/Download-the-latest-source-code-from-GitHub) to acquire the source code.

## Prerequisites

* a basic working knowledge of your Linux distribution. You should know how to install and access your Linux distribution, access the terminal, and install packages from your distribution's package manager.

**Note:** if you are using a virtual machine (VMware, VirtualBox, ...), the compilation process may take a long time. You may want to ensure hardware-assisted virtualization is enabled in your system's BIOS (if applicable) and that your virtual environment is configured to use hardware-assisted virtualization and more than one processor core. Refer to the virtual machine's documentation for details on how to configure these settings.

To compile a firmware image, you will need the following:

* A Linux-based operating environment (either a dedicated machine running Linux, or a virtual machine using [VMware](http://www.vmware.com) or [VirtualBox](https://www.virtualbox.org/)).
* At least 6 GiB free disk space; more is better. This includes the space for packages you may need to install, roughly 1 GiB for the source code and the rest for object files and other intermediate created by the build.

## Preparing your build environment

Before we can build `asuswrt-merlin`, we need to install packages required that will be used during the build process to compile the firmware. Open a terminal window and type or paste in the following:

    sudo apt-get --no-install-recommends install autoconf automake bash bison bzip2 diffutils file flex g++ gawk gcc-multilib gettext gperf groff-base libncurses-dev libexpat1-dev libslang2 libssl-dev libtool libxml-parser-perl make patch perl pkg-config python sed shtool tar texinfo unzip zlib1g zlib1g-dev

When prompted, enter your password for `sudo` to gain `root` access to install the packages. Enter 'Y' when prompted to install the packages. When the packages have finished installing, you will be returned back to your command line.

If you have Debian x64 (amd64) edition you need the extra packages `lib32z1-dev` and `lib32stdc++6`.

```bash
sudo apt-get --no-install-recommends install lib32z1-dev lib32stdc++6
```

Starting debian 7, if you're on amd64, you'll need to setup multiarch gcc, you'll need to add i386 arch to package dependencies, to get libelf1 / libelf-dev 32 bits. to do so:

```bash
sudo dpkg --add-architecture i386
sudo apt-get update
sudo apt-get install libelf-dev:i386 libelf1:i386
```

If you're unsure whether you have Debian x64 or not, use the following command line instead:

```bash
[[ "$(uname -m)" == "x86_64" ]] && sudo apt-get --no-install-recommends install lib32z1-dev lib32stdc++6
```

**Note:** the following commands assume that you have downloaded the source code into `~/asuswrt-merlin/` as specified in the [Download the latest source code from GitHub](https://github.com/RMerl/asuswrt-merlin/wiki/Download-the-latest-source-code-from-GitHub) guide on this wiki. If this is not where you have downloaded the source code to, adjust the commands/paths as necessary for your own environment.

Now we must append to our system's `PATH` environment variable so the build system can find the toolchain required for cross-compilation. First start by creating a symbolic link to make appending easier:

```bash
sudo ln -s ~/asuswrt-merlin/tools/brcm /opt/brcm
```

Now update our `PATH` environment variable (note that you will need to rerun this command each time you open your terminal in future compilations):

```bash
export PATH=$PATH:/opt/brcm/hndtools-mipsel-uclibc/bin
```

## Building the firmware: manual approach

We are now ready to compile the firmware. Keep in mind that this process will take some time, so feel free to grab a coffee or go for a walk while the firmware builds.

When the build completes, the newly created firmware image will be located in the folder you `cd` into for the `make`, i.e. one of:

* `~/asuswrt-merlin/release/src-rt/image/`
* `~/asuswrt-merlin/release/src-rt-6.x/image/`
* `~/asuswrt-merlin/release/src-rt-6.x.4708/image/`

### RT-N16

```bash
cd ~/asuswrt-merlin/release/src-rt
make clean
make rt-n16
```

### RT-N66U & RT-AC66U

```bash
cd ~/asuswrt-merlin/release/src-rt-6.x
make clean
```

#### ... then for RT-N66U

```bash
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

## Cleaning up

As code changes and becomes updated, you may wish to recompile your firmware images to use more current versions of the source code. It's a good idea to clean your build environment to remove leftover files from previous builds and ensure that they won't cause conflicts during a new build.

**Note:** The `make clean` has already been included in the command lines for building above.

If you still insist on cleaning out the directory manually, you can use the following approach:

### RT-N16

```
make -C ~/asuswrt-merlin/release/src-rt clean
```

### RT-N66U & RT-AC66U

```bash
make -C ~/asuswrt-merlin/release/src-rt-6.x clean
```

### RT-AC56U & RT-AC68U

```bash
make -C ~/asuswrt-merlin/release/src-rt-6.x.4708 clean
```

## Additional cleanup

If you have made manual changes or want to reset the Git clone to the state it came in, use:

```bash
git reset --hard
```

from inside the `~/asuswrt-merlin` directory.

## Conclusion

That's it! This guide should have helped you in compiling the `asuswrt-merlin` source code into firmware images suitable for flashing onto your router.

## Automated all-in-one script

**Status:** This has been tested on Debian 7.8 and is known to work as of 2015-02-02.

The latest version can always be found at [assarbad/build-asuswrt-merlin](https://github.com/assarbad/build-asuswrt-merlin). Please report issues there as well.

On the landing page (above link) you will find a brief introduction on how to use the script as well as its status.