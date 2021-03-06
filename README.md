# wdmc-gen2 
WD My Cloud (Gen2) - wdmc-gen2 - Marvell ARMADA 375
===================================================
WD My Cloud  Gen2 (Kernel / Distribution / Information) drop

This is a fork of repo https://github.com/Johns-Q/wdmc-gen2
Previous work was done by Fox: https://fox-exe.ru/WDMyCloud/Other/Official_linux_kernel/
All credits go to the authors if it.

Here is my small updates to get kernel 4.12.9 running on wdmc gen2.
The main change is adding USB sound support so that You can plug-in USB audio adapter
or headset and let wdmc play some mp3's. For this reason a modified .config file is provided
and uImage and libs.tar are pre-built accordingly.

I've used Windows WSL with Ubuntu Bash from Windows Store.
The whole procedure is based on Johns-Q description.

* Install packages for cross-compiling, You will need at least:
	- #sudo apt-get install gcc-arm-linux-gnueabi
    - #sudo apt install u-boot-tools
	- probably You will need to install some more...
* Clone this repo:
	- #git clone https://github.com/alexk195/wdmc-gen2.git
* Get the kernel sources. I used 4.12.9 which is compatible with root-fs I have used (jessie) and 
if You have 4.12 than chances are quite good that kernel update will work with no probs.
	- #wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.12.9.tar.gz
	- Unpack them:
	- #tar -xzf linux-4.12.9.tar.gz
	- Copy the config and dts files:
    - #cp wdmc-gen2/kernel-4.12.9.config linux-4.12.9/.config
    - #cd linux-4.12.9/
    - #cp ../wdmc-gen2/armada-375-wdmc-gen2.dts arch/arm/boot/dts
	
* Now we are going to cross-compile the kernel
	- First thing if You have another version than 4.12.9 is reusing old config:
	- #make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- oldconfig
	- Now You might also want to modify some settings (but it's not required)
	- #make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- menuconfig
	- Cross compiling the kernel:
	- #make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j4 zImage
	- Create DTB file:
	- #make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- armada-375-wdmc-gen2.dtb
	- Create uImage
	- #cat arch/arm/boot/zImage arch/arm/boot/dts/armada-375-wdmc-gen2.dtb > zImage_and_dtb
	- #mkimage -A arm -O linux -T kernel -C none -a 0x00008000 -e 0x00008000 -n 'WDMC-Gen2' -d zImage_and_dtb uImage
	- #rm zImage_and_dtb
* Next we are going to build modules in a local directory. We create a directory for this e.g. install_linux:
	- #mkdir ~/install_linux
	- #make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- INSTALL_MOD_PATH=~/install_linux -j4 modules
	- #make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- INSTALL_MOD_PATH=~/install_linux -j4 modules_install
* Achive the modules in lib for better handling:
	- #cd ~/install_linux/
	- #tar -cf lib.tar lib
* Next step is copying lib.tar and uImage to WDMC. The uImage goes to /boot and modules in lib.tar go to /lib/modules.
  You should untar the lib.tar so that folder 4.12.9 is on WDMC's location /lib/modules/4.12.9.
	- #tar -xf lib.tar
  If You have WDMC with 4.12 running just copy the uImage and untar lib.tar. For fresh system follow instructions from Johns-Q below.
  In the repo You will find the uImage-4.12.9 and modules-4.12.9.tar.gz pre-built.
  
Original Information from Johns-Q:

WD My Cloud (Gen2) - wdmc-gen2 - Marvell ARMADA 375
===================================================

* mainline kernel support
	tested with 4.11.x / 4.12.x
	- device tree source
		armada-375-wdmc-gen2.dts
	- kernel config
		kernel-4.11.8.config, kernel-4.12.0.config
	- kernel uImage with dtb
		uImage (4.12.0)


	- build your own kernel

		- download kernel source from https://www.kernel.org/
		- extract the kernel archive
		- copy kernel-4.12.0.config to linux-4.12/.config 
		- copy armada-375-wdmc-gen2.dts to
		  linux-4.12/linux-4.12/arch/arm/boot/dts/
		- ready to build the kernel (you can use build_kernel_image.sh)
		```
		cd linux-4.12
		make -j2 zImage
		make armada-375-wdmc-gen2.dtb
		cat arch/arm/boot/zImage arch/arm/boot/dts/armada-375-wdmc-gen2.dtb > zImage_and_dtb
		mkimage -A arm -O linux -T kernel -C none -a 0x00008000 -e 0x00008000 -n 'WDMC-Gen2' -d zImage_and_dtb uImage
		rm zImage_and_dtb
		make -j2 modules modules_install

		```
		copy uImage to /boot of your boot partition

	- uRamdisk modified (Original from AllesterFox)

		Can boot original firmware, debian from AllesterFox,
		alpine linux.  It looks for a linux part as 2nd on usb stick
		and boots from there.  If there is none, it boots from 3rd
		partition on SATA drive.  Copy to /boot on 1st partition of
		the usb stick or to 3rd partition of the harddrive.

	- build-initramfs.sh

		builds a minimal initramfs.  Can boot from kernel commandline,
		usb-stick 2nd partition, hdd 3rd partition.
		placed in /boot/uInitrd. rename it to uRamdisk.
		
* [alpine linux](https://alpinelinux.org/) diskless image

	- alpine-wdmc-gen2-3.6.2-armhf.tar.gz

		contains a complete bootable diskless image of alpine linux.
		modified to ignore "root=". "root" is not setable on
		WD My Cloud. And enabled SSH daemon, you can ssh to it after
		booting. *root account has no password.*
		Kernel 4.12 is included, but no modules.

	- alpine initramfs image with ssh (without password!)

* install
	- Use an USB 2.0 stick!
		I had the problem, that usb 3.0 sticks aren't always booting.
		If you have only an usb 3.0 stick, remove stick, turn wdmc2 on,
		when the blue light blinks insert usb stick fast, using this i
		could boot from usb 3.0 sick.

	- make a dos partition table (not gpt) on usb stick.
	- make a partition "W95 FAT32 (LBA)" numeric 0x0C on usb stick
	- make a FAT32 filesystem on usb stick.
	- extact alpine-wdmc-gen2-3.6.2-armhf.tar.gz to the root of the
	  usb stick.

	  You should have:
	  ```  
	  `-rwxr-xr-x    1 root     root            26 Jun 17 11:47 .alpine-release`
	  `-rwxr-xr-x    1 root     root          3427 Jan  1  1980 alpine.apkovl.tar.gz´
	  `drwxr-xr-x    3 root     root          4096 Jun 17 11:47 apks`
	  `drwxr-xr-x    5 root     root          4096 Jul  5 12:02 boot`
          ```
	  on the usb stick.
