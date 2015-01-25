> WARNING DESPITE THE FACT THAT IT IS POSSIBLE TO COMPILE YOUR OWN FIRMWARE,
> I DO NOT ADVISE YOU TO DO SO
> THIS GUIDE IS NOT WRITTEN BY A PROFESSIONAL BUT BY AN AMATEUR<>NOVICE
> THERE ARE A LOT OF THINGS THAT COULD GO WRONG DURING THE PROCESS
> THE AUTHOR OF THIS GUIDE NOR THE SOURCE PROVIDER CAN BE HELD RESPONSIBLE FOR ANY DAMAGE THAT MAY OCCUR BY 
> FLASHING YOUR SELF COMPILED FIRMWARE ON YOUR ROUTER
> IF YOU REALLY WANT TO LEARN THIS TYPE OF COMPUTING AND CODING I SUGGEST YOU FIRST GET FAMILIAR WITH LINUX


But on the other hand your router is VERY hard to brick!



For the reference we are going to use Ubuntu 12.04 in VMware Player.
You can download Ubuntu [Here](http://www.ubuntu.com/download)
And VMware Player [Here](https://my.vmware.com/web/vmware/free#desktop_end_user_computing/vmware_player/5_0)

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

* Fire up terminal CTRL+ALT+T

We are going to make the root account active because if we don't do that it will give us a lot of errors during compilation.

Paste in the following lines in terminal

``` 
sudo passwd root 

```

* It will now ask you for your new ROOT password ( confirm this twice and be sure to remember it we need it ! )

Paste in

```
su
```

As you can see you are now running the terminal in root!

![root](http://members.home.nl/frits.pruymboom/Howto%20Compile%20From%20Source/Root.png)

* We are going to download some packages ( I am sure there are some you don't need, but this is how I got it working and you can delete Ubuntu afterward so who cares right ? )

Paste in

```
sudo apt-get install autoconf automake bash bison bzip2 diffutils file flex m4 \
g++ gawk groff-base libncurses-dev libtool libslang2 make patch perl pkg-config \
shtool subversion tar texinfo zlib1g zlib1g-dev git-core gettext libexpat1-dev \
libssl-dev cvs gperf unzip python libxml-parser-perl gcc-multilib gconf-editor \
libxml2-dev g++-4.4 g++-multilib gitk libncurses5 mtd-utils libncurses5-dev \
libstdc++6-4.4-dev libvorbis-dev g++-4.4-multilib git autopoint autogen sed \
build-essential intltool libelf1:i386 libglib2.0-dev automake-1.11
```
If you have Ubuntu x64 edition then you need `lib32z1-dev` and `lib32stdc++6`
```
sudo apt-get install lib32z1-dev lib32stdc++6
```
If you are ready installing all these packages take a coffee or a beer.

We are going to download merlins hard work with again some terminal commands.  It appears as if scripts published in support forums all assume the clone is done off of root, so first we change to the root directory.

```
cd /root
git clone https://github.com/RMerl/asuswrt-merlin.git
```

Be patient it takes some time

With these command you will build your environment which you will need to work with.
```
sudo ln -s ~/asuswrt-merlin/tools/brcm /opt/brcm
```
```
sudo ln -s ~/asuswrt-merlin/release/src-rt-6.x.4708/toolchains/hndtools-arm-linux-2.6.36-uclibc-4.5.3 /opt/brcm-arm
```
```
export PATH=$PATH:/opt/brcm/hndtools-mipsel-linux/bin:/opt/brcm/hndtools-mipsel-uclibc/bin:/opt/brcm-arm/bin
```

```
sudo mkdir -p /media/ASUSWRT/
```

```
sudo ln -s ~/asuswrt-merlin /media/ASUSWRT/asuswrt-merlin
```

We are now ready to build a image

* For RT-N16

```
cd ~/asuswrt-merlin/release/src-rt
```

```
make clean
```

```
make rt-n16
```

* For RT-N66U

```
cd ~/asuswrt-merlin/release/src-rt-6.x
```

```
make clean
```

```
make rt-n66u
```


* For RT-AC66U



```
cd ~/asuswrt-merlin/release/src-rt-6.x
```

```
make clean
```
```
make rt-ac66u
```
* For RT-AC56U

```
cd ~/asuswrt-merlin/release/src-rt-6.x.4708
```

```
make clean
```

```
make rt-ac56u
```
* For RT-AC68U

```
cd ~/asuswrt-merlin/release/src-rt-6.x.4708
```

```
make clean
```

```
make rt-ac68u
```

**Notes for Ubuntu 13.10:**

If you want to build in Ubuntu 13.10, before you make clean / make [router], you might need to perform these steps due to the different version of autoconf.

This works to build 3.0.0.4_374.38_1.

```
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

## All in one script

<span style="color:red">**WARNING:**</span> currently this is hardcoded to use the path `release/src-rt-6.x`. I will take care of this shortly.

Last tested on `master` as of 2015-01-25. The script takes care of the steps mentioned above for Ubuntu 13.10 and newer. In addition it does fixups to the source tree which ensure that it can run without superuser rights as long as the user makes has all the packages installed. It will also check for the packages being installed and bail out with a meaningful error message if not.

It can be run in two modes: with or without `sudo` involved. With `sudo` it will use the symbolic link method inside `/opt`, whereas without it will modify the files in the source tree to adjust hardcoded paths.

If you are running inside a [`tmux`](http://tmux.sourceforge.net/) pane, this will also pipe the output into a log file.

### Syntax

* Unprivileged: `./ubuntu-build-image <router-model> [path-to-asuswrt-merlin]`
* With `sudo`: `USE_SUDO=1 ./ubuntu-build-image <router-model> [path-to-asuswrt-merlin]`

You can leave out the `path-to-asuswrt-merlin` argument and the script will check the folder it resides in for a marker file (`README-merlin.txt`) to see whether this is the expected source tree.

Any non-empty value for `USE_SUDO` can be used and you can of course also export it up front, instead of prepending it to the command line.

### The script

```bash
#!/usr/bin/env bash
# vim: set autoindent smartindent ts=4 sw=4 sts=4 noet filetype=sh:
###############################################################################
## Based on instructions from:
## https://github.com/RMerl/asuswrt-merlin/wiki/Compiling-from-source-using-a-Debian-based-Linux-Distribution
## https://github.com/RMerl/asuswrt-merlin/wiki/Compile-Firmware-from-source-using-Ubuntu
###############################################################################
[[ -n "$DEBUG" ]] && set -x
pushd $(dirname $0) > /dev/null; CURRABSPATH=$(readlink -nf "$(pwd)"); popd > /dev/null
[[ -t 1 ]] && { G="\e[1;32m"; R="\e[1;31m"; Z="\e[0m"; B="\e[1;34m"; W="\e[1;37m"; Y="\e[1;33m"; }
ROUTERMDL=${1}
BASEDIR=${2:-$CURRABSPATH}
CURRDATE=$(date +'%Y-%m-%dT%H-%M-%S')
_TMUX_LOG="$BASEDIR/tmux-${CURRDATE}.log"
help_syntax()
{
	echo -e "${W}SYNTAX:${Z} ${0##*/} <router-model> [path]\n"
	echo -e "\twhere:\n\
\t* ${W}path${Z} is a directory containing the asuswrt-merlin sources\n\
\t  (default is the directory with this script: $CURRABSPATH)\n\
\t* ${W}router-model${Z} is the model name you'd pass to make\n\
\t  (e.g. rt-n66u, rt-ac66u etc)\n"
	echo -e "Variables:\n\t${W}USE_SUDO${Z} if set will 'install' and use tools in /opt"
}
fatal()
{
	[[ "x$1" == "xhelp" ]] && { SHOWHELP=1; shift; }
	echo -e "${R}ERROR:${Z} ${1:-<unknown>}"
	((SHOWHELP)) && help_syntax
	exit 1
}
cleanup()
{
	if [[ -n "$TMUX_PANE" ]]; then
		echo -e "${Y}CLEANUP${Z}: turning off tmux logging ($_TMUX_LOG)"
		tmux pipe-pane -t $TMUX_PANE
	fi
}
if [[ -n "$TMUX_PANE" ]]; then
	echo -e "${Y}NOTE:${Z} turning on tmux logging ($_TMUX_LOG)"
	tmux pipe-pane -t $TMUX_PANE "cat >> $_TMUX_LOG" && trap cleanup EXIT
fi
[[ -n "$ROUTERMDL" ]] || fatal help "you have to ${W}give the router model${Z} to build for."
[[ -f "$BASEDIR/README-merlin.txt" ]] || fatal help "$BASEDIR is expected to contain a file ${W}README-merlin.txt${Z}."
[[ -e "/etc/debian_version" ]] || fatal "this script expects the Debian flavor called ${W}Ubuntu${Z}."
type lsb_release > /dev/null 2>&1 || fatal "${W}lsb_release{$Z} is expected to be installed. That is normally the case on Ubuntu anyway."
[[ "xUbuntu" == "x$(lsb_release -si)" ]] || fatal "this script has only been tested on ${W}Ubuntu${Z}."
let UBUREL=0x$(lsb_release -sr|tr -d '.')
((UBUREL >= 0x1204)) || fatal "expecting a ${W}Ubuntu >= 12.04{$Z} (precise)."
((UBUREL == 0x1404)) || echo -e "${Y}NOTE:${Z} this script has only been tested on Ubuntu 14.04. If you run it on a different platform, please report back if it works. But more importantly report back if it doesn't. Thanks."
# All those packages are required according to the asuswrt-merlin docs
PREREQ_PKGS="autoconf automake bash bison bzip2 diffutils file flex g++ gawk gcc-multilib gettext gperf groff-base libncurses-dev libexpat1-dev libslang2 libssl-dev libtool libxml-parser-perl make patch perl pkg-config python sed shtool tar texinfo unzip zlib1g zlib1g-dev lib32z1-dev lib32stdc++6"
# On Ubuntu 13.10 and later we check for two additional packages
((UBUREL >= 0x1310)) && PREREQ_PKGS="$PREREQ_PKGS automake1.11 libproxy-dev"
echo -en "${R}"
dpkg-query --load-avail -l $PREREQ_PKGS > /dev/null || fatal "it appears you have packages missing (see above) that are required to build a firmware image.\nUse\n\tapt-get --no-install-recommends install ...\nto install the missing packages, then retry."
echo -en "${Z}"
# Special steps required on Ubuntu newer than or equal to 13.10
if ((UBUREL >= 0x1310)); then
	echo -e "${Y}NOTE:${Z} Attempting fixes for Ubuntu >= 13.10"
	# fix neon missing proxy.h
	[[ -f "/usr/include/proxy.h" ]] || fatal "expected to find proxy.h from package libproxy-dev in /usr/include. It's not there, though."
	cp /usr/include/proxy.h "$BASEDIR/release/src/router/neon/" || fatal "failed to copy proxy.h into source tree" 
	# fix broken configure script for libdaemon
	( cd "$BASEDIR/release/src/router/libdaemon" && aclocal ) || fatal "failed to fix broken configure script for libdaemon"
	# fix broken configure script for libxml2
	( cd "$BASEDIR/release/src/router/libxml2" && sed -i s/^AM_C_PROTOTYPES/#AM_C_PROTOTYPES/g configure.in && aclocal ) || fatal "failed to fix broken configure script for libxml2"
fi
# Clean up leftovers
echo -e "${Y}NOTE:${Z} removing excess files"
rm -f "$BASEDIR/release/src/router/".#preconfigure*
if [[ -n "$USE_SUDO" ]]; then
	echo -e "${Y}NOTE:${Z} making toolchain available in /opt"
	sudo ln -sf "$BASEDIR/tools/brcm" /opt/brcm
	export TOOLCHAIN="/opt/brcm/hndtools-mipsel-uclibc"
else
	# Cheapo version of "installing" the toolchain to /opt/brcm
	echo -e "${Y}NOTE:${Z} looking for files in which to fix hardcoded path to toolchain"
	grep -RP '=\s*/opt/brcm/hndtools-mipsel-uclibc/' "$BASEDIR/release/" 2>/dev/null|cut -d : -f 1|sort -u|while read fname; do
		[[ -e "${fname}.BAK-build-image" ]] && { echo -e "${Y}NOTE:${Z} skipping fixup of ${fname} (prior run detected)"; continue; }
		[[ "x${fname//.BAK-build-image/}" == "x$fname" ]] || continue # skip backup files
		echo -e "Rebasing paths in: ${G}${fname}${Z}"
		sed -i.BAK-build-image 's#/opt/brcm/hndtools-mipsel-uclibc/#'"$BASEDIR/tools/brcm/hndtools-mipsel-uclibc/"'#g' "$fname" && grep --color=auto "$BASEDIR/tools/brcm/hndtools-mipsel-uclibc/" "$fname"
	done
	# Instead of installing, we override some of the paths used in the build process
	export TOOLCHAIN="$BASEDIR/tools/brcm/hndtools-mipsel-uclibc" # used in preconfigure scripts
fi
# Add the toolchain into the PATH
export PATH="$PATH:$TOOLCHAIN/bin"
# Check that the relevant tools execute from there
for i in mipsel-linux-addr2line mipsel-linux-ar mipsel-linux-as mipsel-linux-c++ mipsel-linux-cc mipsel-linux-c++filt mipsel-linux-cpp mipsel-linux-g++ mipsel-linux-gcc mipsel-linux-gcc-4.2.4 mipsel-linux-gccbug mipsel-linux-gcov mipsel-linux-gprof mipsel-linux-ld mipsel-linux-nm mipsel-linux-objcopy mipsel-linux-objdump mipsel-linux-ranlib mipsel-linux-readelf mipsel-linux-size mipsel-linux-strings mipsel-linux-strip; do
	$i --version > /dev/null 2>&1 || fatal "failed to execute $i"
done
make -C "$BASEDIR/release/src-rt-6.x" "TOOLCHAIN=$TOOLCHAIN" clean > /dev/null 2>&1
[[ "clean" == "$ROUTERMDL" ]] || make -C "$BASEDIR/release/src-rt-6.x" "TOOLCHAIN=$TOOLCHAIN" $ROUTERMDL
```