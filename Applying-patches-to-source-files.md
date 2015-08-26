Before building the firmware, you may apply patches to source files

Usage: **patch -p1 < /path/file.patch** but first, navigate to root of directory source

Don't forget to change "router" in paths with your ubuntu username

***


For example [this](http://sourceforge.net/tracker/download.php?group_id=243163&atid=1121516&file_id=457910&aid=3596625) patch fixes minidlna bookmarks on Samsung E Series TV.
```
cd /home/router/Downloads/ 

wget "http://sourceforge.net/tracker/download.php?group_id=243163&atid=1121516&file_id=457910&aid=3596625" -O samsungES.patch

cd /home/router/asuswrt-merlin/release/src-rt/router/minidlna/

patch -p1 < /home/router/Downloads/samsungES.patch
```

Another example of [patch](http://dl.dropbox.com/u/47669650/RT-N66U/patches/sonybdp-sx90.patch.zip) that solve streaming mkv files on Sony blu-ray players
```
cd /home/router/Downloads/ 

wget http://tinyurl.com/sonybdp-sx90-patch-zip

unzip sonybdp-sx90-patch-zip

cd /home/router/asuswrt-merlin/release/src-rt/router/minidlna/

patch -p1 < /home/router/Downloads/sonybdp-sx90.patch 
```

***

Now we are ready to [build the firmware](https://github.com/RMerl/asuswrt-merlin/wiki/Compiling-from-source-using-a-Debian-based-Linux-Distribution).