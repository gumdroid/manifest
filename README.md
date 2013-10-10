Build Android for Gumstix Boards
================================

This repository provides Repo manifests to build Android for Gumstix products.

***
**Note:**
If you already have an Android build setup and just want the device
directories, you can find them here:
**Pepper**: git://github.com/ashcharles/pepper.git
**Overo**: git://github.com/ashcharles/overo.git
***

Repo is a tool that enables the management of many git repositories given a 
single *manifest* file.  Tell repo to fetch a manifest from this repository and
it will fetch the git repositories specified in the manifest and, by doing so,
setup an Android build environment for you!

####TOC####

- [Build Android for Gumstix Boards](#build-android-for-gumstix-boards)
	- [Getting Started](#getting-started)
			- [1.  Configure Development machine.](#1--configure-development-machine)
			- [2.  Install Repo.](#2--install-repo)
			- [3.  Initialize a Repo client.](#3--initialize-a-repo-client)
			- [4.  Fetch all the repositories](#4--fetch-all-the-repositories)
			- [5. Build Android](#5-build-android)
			- [6. Create micro SD card](#6-create-micro-sd-card)
			- [7. Boot Android](#7-boot-android)
			- [8. Use Android](#8-use-android)
			- [9. Issues](#9-issues)
			- [10. Resources](#10-resources)

Getting Started
---------------
####1.  Configure Development machine.####

**Note:**
Android requires a 64-bit build machine.  These instructions have been tested on an Ubuntu 12.10 installation but recent Ubuntu releases should work.  See http://source.android.com/source/initializing.html for more details.
**

Android only likes official Java.

    $ sudo add-apt-repository ppa:webupd8team/java
    $ sudo apt-get update && sudo apt-get install oracle-java6-installer

Grab some other packages.

    $ sudo apt-get install git gnupg flex bison gperf build-essential zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386 uboot-mkimage
    $ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so

(Optional) To use ADB (the Android Debug Bridge), give access to the USB port.

    $ sudo sh -c "echo 'SUBSYSTEM==\"usb\", ATTR{idVendor}==\"18d1\", ATTR{idProduct}==\"d002\", MODE=\"0666\"' >> /etc/udev/rules.d/51-android.rules"

####2.  Install Repo.####

Download the Repo script:

    $ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > repo

Make it executable:

    $ chmod a+x repo

Move it on to your system path:

    $ sudo mv repo /usr/local/bin/

If it is correctly installed, you should see a Usage message when invoked
with the help flag.

    $ repo --help

####3.  Initialize a Repo client.####

Create an empty directory to hold your working files:

    $ mkdir android 
    $ cd android

Tell Repo where to find the manifest:

    $ repo init -u git://github.com/gumdroid/manifest.git -b gumdroid-4.3_r3.1


A successful initialization will end with a message stating that Repo is
initialized in your working directory. Your directory should now
contain a .repo directory where repo control files such as the manifest are
stored but you should not need to touch this directory.
***
**Note:**
You can use the **-b** switch to specify the branch of the repository to use.

The **-m** switch selects the manifest file (default is *default.xml*).

To test out the bleeding edge, type:

    $ repo init -u git://github.com/gumdroid/manifest.git -b dev
    $ repo sync

To get back to the known stable version, type:

    $ repo init -u git://github.com/gumdroid/manifest.git -b master
    $ repo sync

To learn more about repo, look at http://source.android.com/source/version-control.html 
***

####4.  Fetch all the repositories####

    $ repo sync

Now go turn on the coffee machine as this may take an hour or so depending on
your connection.

***
####5. Build Android####

There are 4 make targets to create a functional Android image: bsp systemtarball boottarball userdatatarball.
Building these components is straightforward. However interdependency between bsp and systemtarbal targets requires a workaround at the moment. 

First build systemtarball

    $ make TARGET_PRODUCT=pepper -j8 systemtarball:

Now build the remaining targets while updating systemtarball:

    $ make TARGET_PRODUCT=pepper -j8 bsp systemtarball boottarball userdatatarball

***
####6. Create micro SD card####

In the build output directory `out/target/product/pepper`, you will find these files necessary to build a bootable micro SD card.

    ./boot
    ├── cmdline
    ├── MLO
    ├── u-boot.img
    ├── uEnv.txt
    ├── uImage
    └── uInitrd
    ./boot.tar.bz2 
    ./system.tar.bz2 
    ./userdata.tar.bz2 

 There are 4 paritions required: boot, system, cache, and data. You can use mkandroidsd.sh. For example,
 
    $ ./mkandroidsd.sh /dev/sdh  

***
####7. Boot Android####

If Pepper is booting but the touch screen remains dark, try the following in the u-boot command line:

    $ i2c mw 0x24 0x7 0x8
    $ i2c mw 0x24 0x8 0x63
    $ boot

Then in shell through the consol serial port, 

    $ ./display-enable.sh

Animated Android boot image should appear on the screen before you see the Android lock screen. 

***
####8. Use Android####

* **Push buttons on Pepper act similarly to the hardware buttons of other Android devices:**

    Switch 2 (S2/GPIO54) is configured as the back button. 
    Switch 3 (S3/GPIO55) is configured as the menu button.

   Additionally, power button can be added to available `ECAP_IN_PWM0_OUT` (Pin 8 in the 40 pin header). 

* **2 Micro USB are configured as the following:**

    USB 1 (J4) is a slave. You can access Android Debug Bridge (ADB) with this.
    USB 2 (J5) is a host. Using a micro USB to USB OTG Host Adapter, you can attach peripherals such as keyboards and mouse. 

* **Suppress I2C bus messages:**

   There will be peiriodic output to the serial console like this:

        [ 72.078553] omap_i2c omap_i2c.1: timeout waiting for bus ready
        [ 76.758463] omap_i2c omap_i2c.1: controller timed out

   You can suppress this by adding 1.5K ohm pullup resistors from `VDD_3V3C` (pin 26) to `I2C0_SCL` (pin 33)  and also to `I2C0_SDA` (pin 31) in the 40 pin header.

***
####9. Issues####
The following peripherals are not working at the moment:

* **Marvell mwifiex SD8787**

   This multi-role chip requires a high level of integration between kernel and userspace. Getting **Wifi** working involves configurations related to nl80211, cfg80211, and etc. 
   The device is failing to come up in such manner: 
   
        W/CommandListener( 82): Failed to retrieve HW addr for wlan0 (No such device)
        D/CommandListener( 82): Settin[ 33.667590] init: no such service 'wpa_supplicant'

   Network device management in Android seems to be the culprit as this device works well in Yocto images. Settings are mostly found in the kernel configuration file and in the BoardConfig.mk (in *device/gumstix/pepper/*). The driver binary (*sd8787_uapsta.bin*) can be found in the Pepper device directory (*device/gumstix/pepper/*). 

   Additionally **Bluetooth** has to be configured separately. Since Android Jellybean, Google switched to Broadcom's Bluedroid from Bluez. The new bluetooth stack does not support bluetooth devices using UART. Possible workarounds include writing a custom bluetooth profile to work with Bluedroid and swapping back Bluez entirely.  

* **TI TLV320AIC3106**

   The audio interface is getting detected but there is no audible output from the jack.

        root@pepper:/ # ls proc | grep asound                                          
        asound
        root@pepper:/data/media/0 # ls /dev/snd/p
        pcmC0D0c  pcmC0D0p  
        root@pepper:/data/media/0 # tinyplay M1F1-int16-AFsp.wav  -d 0                 
        Unable to open PCM device 0 (cannot set hw params: Invalid argument)

   On Gumstix Yocto image for Pepper, the line-out works (albeit the volume is small). A good debug point would be the hardware application layer (HAL) for audio which is found in *device/gumstix/pepper/libaudio*. Another possible fail point would be the mixer settings. Although Android uses ALSA just like the Yocto image, it uses *tinymix*. 

* **Ethernet**

####10. Resources ####
* Gumstix Developer Center - http://gumstix.org
* Gumstix Mailing List - https://lists.sourceforge.net/lists/listinfo/gumstix-users
* Pepper board schematic - http://pubs.gumstix.com/boards/PEPPER/R4021/
* Gumstix Android repository - https://github.com/gumdroid/
