# Compiling the kernel, installing kernel modules and making a JFFS2 rootfs for the Friendlyarm 64M dev board #

## Download toolchain, u-boot and kernel ##

Go to:
> http://code.google.com/p/mini2440/downloads/list and download:

> `mini2440-bootstrap-v2.sh`

Study this script.

In your `[home]` directory do a <Save file> from the Wiki Downloads page:

> http://friendlyarm.googlecode.com/files/file-download.sh

and run it:

> `sudo ./file-download.sh`

This script will get you the toolchain, the latest kernel, u-boot, compile u-boot and set-up the required directories.  Note: there are some issues with u-boot, please see other posts on resolving this issue.  I use an old version  `u-boot.bin`  as I am running a 64M dev board.

## Place in .profile on your Linux host ##

  * export PATH=/home/user/mini2440-bootstrap/arm-2008q3/bin:$PATH

  * CROSS\_COMPILE=arm-none-linux-gnueabi-
  * CC=”${CROSS\_COMPILE}gcc –march=armv4t –mtune=arm920t”
  * export CROSS\_COMPILE
  * export CC

Depending on permission’s  **sudo**  may be required on some commands.

## Compile the kernel ##
(cd to mini2440-bootstrap)

```
[mini2440-bootstrap] # cd kernel/mini2440
[mini2440] # make ARCH=arm O=../../kernel-bin/ mrproper
[mini2440] # make ARCH=arm O=../../kernel-bin/ mini2440_defconfig
[mini2440] # make ARCH=arm O=../../kernel-bin/ menuconfig
```
At this point `<Load an alternative configuration file>` , for example `../config_mini2440_n35` . N.B. this will create a YAFFS2 file system.
  * configure the kernel to support the wanted file systems and MTD
  * select built-in and loadable kernel modules

When finished making changes `<Save as an alternative configuration file>` with the name `.config` .  Select `<Exit>` and the compilation should proceed.
When compilation finishes you can save your .config file as something unique for later use.

```
[mini2440] # make ARCH=arm O=../../kernel-bin/ -j4
[mini2440] # make ARCH=arm O=../../kernel-bin/ -j4 modules
```

If this fails and/or you want to start from a clean base do a  `make ARCH=arm distclean` .

> now, install the kernel modules

```
[mini2440] # sudo make ARCH=arm O=../../kernel-bin/ INSTALL_MOD_PATH=/home/user/mini2440-bootstrap/target/ modules_install
```

> strip debug info, if you are worried about the size

```
# cd /home/user/mini2440-bootstrap/target/lib/modules

[modules] # /home/user/mini240-bootstrap/arm-2008-q3/arm-none-linux-gnueabi/bin/strip `find . –name “*.ko”`
```

> (note: two back ticks)

## Change target directory file permission’s ##

```
[mini2440-bootstrap] # sudo chown -R 0:0 /home/user/mini2440-bootstrap/target
```

Makes all files in  `[target]`  owned by root.

## Make the uImage ##

```
[mini2440-bootstrap] # sudo ./uboot/mini2440/tools/mkimage –A arm –O linux -T kernel -C none –a 0x30008000 –e 0x30008000 -d kernel-bin/arch/arm/boot/zImage output/uImage
```


## Compile Busybox ##

I started by compiling Busybox as static.  When things are working properly you may want to re-compile as dynamic.

```
[busybox-1.13.3] # make defconfig
[busybox-1.13.3] # make ARCH=arm menuconfig
[busybox-1.13.3] # make clean
[busybox-1.13.3] # make ARCH=arm
[busybox-1.13.3] # make ARCH=arm CONFIG_PREFIX=/home/user/mini2440-bootstrap/target install 
```

## Configure the New Target Root File System ##

Following the tutorial:
http://wiki.davincidsp.com/index.php/Creating_a_Root_File_System_for_Linux_on_OMAP35x

check that the following are in `[target]` :

```
[target] # ls -la
```

> bin, linuxrc-> bin/busybox, sbin and usr

Create the following directories:

```
[target] # mkdir –p dev
[target] # mkdir –p etc
[target] # mkdir –p lib
[target] # mkdir –p mnt
[target] # mkdir –p opt
[target] # mkdir –p proc
[target] # mkdir –p sys
[target] # mkdir –p tmp
[target] # mkdir –p var
[target] # mkdir –p var/log
[target] # mknod dev/null c 2 2  (chmod 777)
[target] # mknod dev/console c 5 1 (chmod 600)
```

I suspect that the two mknod entries are not required as mdev.conf (in the following example) seems to either create these or over-write them.

Use the /etc directory from this example rootfs.

http://friendlyarm.googlecode.com/files/rootfsjffs2.gz

Note: /etc gets mounted into ram so don’t expect any changes you make to files in /etc to survive a re-boot!

Don't use anything else from this file.  There are probably much better ways to make a rootfs.  Have a look here:

http://rootfs.wordpress.com/2010/04/23/the-busybox-file-system/

In fact he has several good tutorials.

## Add the Shared Libraries Applications will Require ##

```
[mini2440-bootstrap] # cd /target
[lib] # cp –r /home/user/mini2440-bootstrap/arm-2008q3/arm-none-linux-gnueabi/libc/lib/* .
```

The libraries in this folder are for v5TE.  For the v4T architecture I think the files wanted are:

```
[mini2440-bootstrap] # cd /target
[lib] # cp –r /home/user/mini2440-bootstrap/arm-2008q3/arm-none-linux-gnueabi/libc/armv4t/lib/* .
```


## Creating a JFFS2 Root File System ##

Get mtd-tools and install them:

```
[mini2440-bootstrap] #sudo apt-get install mtd-tools
```

Make your rootfs.jffs2 image

```
[mini2440-bootstrap] # mkfs.jffs2 –lqnp –e 16 –r target –o output/my_rootfs.jffs2
```

## Verify your rootfs on the host ##

Check my\_rootfs.jffs2 by running the following script (mount\_jffs2.sh):

> N.B. ensure you have Linux line endings if doing a copy/paste!

### Shell script to mount/unmount JFFS2 using kernel memory emulating MTD ###

```
#!/bin/sh
 JFFSIMG=$1 # jffs image
 MP="/media/jffs2" # mount point
 MTDBLOCK="/tmp/mtdblock0" # MTD device file
 UMNT=""

 echo "$0" | grep unmount_ >/dev/null 2>&1
 [ $? -eq 0 ] && UMNT=1
 if [ $# -gt 1 -a x"$2"x = x"unmount"x ]; then
   UMNT=1
 fi

 if [ x"${UMNT}"x = x""x ]; then
   if [ ! -b ${MTDBLOCK} ] ; then
     mknod ${MTDBLOCK} b 31 0 || exit 1
   fi
   modprobe mtdblock
   modprobe mtdram total_size=65536 erase_size=256
   modprobe jffs2
   dd if=${JFFSIMG} of=${MTDBLOCK}
   [ ! -d ${MP} ] && mkdir -p ${MP}
   mount -t jffs2 ${MTDBLOCK} ${MP}
 else
   umount ${MP}
   if [ $? -ne 0 ]; then
     echo "Cannot unmount JFFS2 at $MP" && exit 1
   fi
   modprobe -r jffs2
   modprobe -r mtdram
   modprobe -r mtdblock
 fi 
```

Place this script in the `[mini2440-bootstrap]` directory.

Make the shell script executable:

```
chmod a+x mount_jffs2.sh
```

Usage: (from `[mini2440-bootstrap]`)

```
sudo ./mount_jffs2.sh output/my_rootfs.jffs2
```

You can also use this script to unmount and unload the non-utilized kernel modules:

```
sudo ./mount_jffs2.sh output/my_rootfs.jffs2 unmount
```

The above script was found here:

http://wiki.maemo.org/Modifying_the_root_image

## Now follow Forrest Bao’s tutorial at: ##

http://narnia.cs.ttu.edu/drupal/node/131

I used the u-boot.bin referenced here `u-boot.bin`.

Substitute your own kernel image.

After you have loaded the kernel you can try the following test:

```
nand read 0x31000000 kernel 0x500000
bootm 0x31000000
```

You should observe that the kernel gets further through the boot process, possibly to the place where the file system would try to mount.

Now add your my\_rootfs.jffs2 image.