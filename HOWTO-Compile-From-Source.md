For the reference we are going to use Ubuntu 12.04 in VMware Player.
You can download Ubuntu [Here](http://www.ubuntu.com/download)
And VMware Player [Here](https://my.vmware.com/web/vmware/free#desktop_end_user_computing/vmware_player/5_0)

When you have both install VMware Player.

Follow these steps :

* Create a New Virtual Machine
* Select Installer disc image file (iso) ( VMware Player will tell you something about easy install, that's okay )
* Give it about 20 GB afterwards you can delete your virtual Ubuntu anyway.
* Crank up some specs give it some more ram and proccesing power, It doesn't sound as a heavy work load but making your .trx image for the router actually takes a long time on a slow proccesor
* When installing Ubuntu it asks for a username its the best if you just name it ''router'' without the quotes because then you can just copy and paste from this guide no need to make everything complicated.

When the Ubuntu installer has finished you can fire up terminal

* Fire up terminal CTRL+ALT+T

We are going to make the root account active because if we don't do that it will give us a lot of error during compilation.

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


* Were are going to download some packages ( I am sure there are some you don't need, but this is how i got it working and you can delete Ubuntu afterward so who cares right ? )

Paste in

```
sudo apt-get install autoconf automake bash bison bzip2 diffutils file flex g++ gawk groff-base libncurses-dev libtool libslang2 make patch perl sed shtool subversion tar texinfo zlib1g zlib1g-dev git-core gettext libexpat1-dev libssl-dev cvs gperf unzip python libxml-parser-perl gcc-multilib gconf-editor gperf synaptic
```

* I made you install SYNAPTIC at the end ( that's because i think synaptic is easy to use )


Go ahead an fire up synaptic

```
synaptic
```

In Synaptic you are going to search for :

```
	bison

	flex

	g++

	g++-4.4

	g++-multilib

	gawk

	gcc-multilib

	gconf (or gconf-editor)

	gitk

	lib32z1-dev

	libncurses5

	libncurses5-dev

	libstdc++6-4.4-dev

	libtool

	m4

	pkg-config

	zlib1g-dev

	gperf

	lib32z1-dev
```

* Installing these should be quite straight forward
* NOTE: Some of these packages are already installed skip them !

If you are ready installing all these packages take a coffee or a beer.

Were are going to download merlins hard work with again some terminal commands

```
git clone https://github.com/RMerl/asuswrt-merlin.git
```

Be patient it takes some time

Now if you have read very carefully i told you to give your account the name ''router''

If you didn't then these command will not work you will have to adjust it to your own name.
With these command you will build your environment which you will need to work with.
```
sudo ln -s /home/router/asuswrt-merlin/tools/brcm /opt/brcm
```

```
export PATH=$PATH:/opt/brcm/hndtools-mipsel-linux/bin:/opt/brcm/hndtools-mipsel-uclibc/bin
```

```
sudo mkdir -p /media/ASUSWRT/
```

```
sudo ln -s /home/router/asuswrt-merlin /media/ASUSWRT/asuswrt-merlin
```

We are now ready to build a image

* For RT-N66U

```
cd /home/router/asuswrt-merlin/release/src-rt
```

```
make clean
```

```
make rt-n66u
```

* For RT-AC66U

```
sudo mkdir /home/router/asuswrt-merlin/release/src-rt-6.x/wl/sysdeps/default/linux
```

```
cd /home/router/asuswrt-merlin/release/src-rt-6.x
```

```
make clean
```
```
make rt-ac66u
```
