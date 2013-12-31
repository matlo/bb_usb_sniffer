bb_usb_sniffer
==============

Tools to set up a usb sniffer on a beagleboard xM.  
This is based on the work of Nicolas Boichat.

Sources  
-------

http://elinux.org/BeagleBoard/GSoC/2010_Projects/USBSniffer  
http://downloads.angstrom-distribution.org/demo/beagleboard/  
http://beagleboard.org/linux  

Set up the sniffer quickly
--------------------------

```
cd ~  
wget http://gimx.fr/download/bb_sniffer_small.img.7z  
7zr e bb_sniffer_small.img.7z  
sudo dd if=bb_sniffer_small.img of=/dev/sdX  
```
(where X is the device letter of the sd card, e.g. /dev/sdc, /dev/sdd...)  
Before removing the sd card, make sure to run sync.

Sniff!
------

Read the instructions at http://elinux.org/BeagleBoard/GSoC/2010_Projects/USBSniffer#Usage_instructions.  
Note that you don't need a USB hub for low/full speed devices if you use a beagleboard xM, because there's one on the board.  

Set up the sniffer from scratch
-------------------------------

1. Clone this git repo:  
```
cd ~  
git clone git://github.com/matlo/bb_usb_sniffer.git
```

2. Format the sd card:  
```
cd ~/bb_usb_sniffer/sdcard  
sudo sh mkcard.txt /dev/sdX  
```
(where X is the device letter of the sd card e.g. /dev/sdc, /dev/sdd...)  
```
sudo mkdir -p /media/bb/boot /media/bb/Angstrom  
sudo mount /dev/sdX1 /media/bb/boot  
sudo mount /dev/sdX2 /media/bb/Angstrom
```

3. Install the bootloader:  
```
cd ~/bb_usb_sniffer/sdcard  
wget http://downloads.angstrom-distribution.org/demo/beagleboard/MLO  
wget http://downloads.angstrom-distribution.org/demo/beagleboard/u-boot.img  
wget https://raw.github.com/matlo/bb_usb_sniffer/master/sdcard/uEnv.txt  
sudo cp MLO u-boot.img uEnv.txt /media/bb/boot
```

4. Copy the rootfs:  
```
cd ~/bb_usb_sniffer  
wget http://downloads.angstrom-distribution.org/demo/beagleboard/Angstrom-Beagleboard-demo-image-glibc-ipk-2011.1-beagleboard.rootfs.tar.bz2  
sudo tar xjvf Angstrom-Beagleboard-demo-image-glibc-ipk-2011.1-beagleboard.rootfs.tar.bz2 -C /media/bb/Angstrom
```

5. Get a toolchain:  
```
mkdir ~/bb_usb_sniffer/tools  
cd ~/bb_usb_sniffer/tools  
wget http://www.angstrom-distribution.org/toolchains/angstrom-2011.03-x86_64-linux-armv7a-linux-gnueabi-toolchain.tar.bz2  
sudo tar xjvf angstrom-2011.03-x86_64-linux-armv7a-linux-gnueabi-toolchain.tar.bz2 -C /
```

6. Compile the kernel:  
```
cd ~/bb_usb_sniffer  
git clone git://gitorious.org/beagleboard-usbsniffer/beagleboard-usbsniffer-kernel.git  
cd beagleboard-usbsniffer-kernel  
patch -p1 < ../patches/xmc.patch  
sudo apt-get install u-boot-tools  
make ARCH=arm CROSS_COMPILE=/usr/local/angstrom/arm/bin/arm-angstrom-linux-gnueabi- uImage  
sudo cp arch/arm/boot/uImage /media/bb/Angstrom/boot/uImage
```

7. Compile and install the modules:  
```
cd ~/bb_usb_sniffer/beagleboard-usbsniffer-kernel  
make ARCH=arm CROSS_COMPILE=/usr/local/angstrom/arm/bin/arm-angstrom-linux-gnueabi- modules  
sudo make ARCH=arm INSTALL_MOD_PATH=/media/bb/Angstrom modules_install
```

8. Copy libpcap and tcpdump packages:  
```
cd ~/bb_usb_sniffer  
wget http://feeds.angstrom-distribution.org/feeds/2011.03/ipk/glibc/armv7a/base/tcpdump_4.1.1-r1.5_armv7a.ipk  
wget http://feeds.angstrom-distribution.org/feeds/2011.03/ipk/glibc/armv7a/base/libpcap_1.1.1-r1.6_armv7a.ipk  
sudo cp tcpdump_4.1.1-r1.5_armv7a.ipk libpcap_1.1.1-r1.6_armv7a.ipk /media/bb/Angstrom/home/root
```

9. Copy the helper-scripts:  
```
sudo cp ~/bb_usb_sniffer/helper-scripts/* /media/bb/Angstrom/home/root
```

10. Final steps:  

* Before removing the sdcard, make sure to run sync.  
* Boot angstrom and login as root.  
* Run depmod -a on the beagleboard for the kernel to find the modules.  
* Install libpcap and tcpdump:  
```
opkg install libpcap_1.1.1-r1.6_armv7a.ipk tcpdump_4.1.1-r1.5_armv7a.ipk  
rm libpcap_1.1.1-r1.6_armv7a.ipk tcpdump_4.1.1-r1.5_armv7a.ipk
```
