Build Android for Gumstix Boards
================================

This repository provides Repo manifests to build Android for Gumstix products.

***
**Note:**
If you already have an Android build setup and just want the device
directories, you can find them here:

 * **Pepper**: git://github.com/gumdroid/pepper.git
 * **Overo**: git://github.com/gumdroid/overo.git
***

Repo is a tool that enables the management of many git repositories given a 
single *manifest* file.  Tell repo to fetch a manifest from this repository and
it will fetch the git repositories specified in the manifest and, by doing so,
setup an Android build environment for you.

##Contents##

 * [Configure Development Machine](#1--configure-development-machine)
 * [Install Repo](#2--install-repo)
 * [Initialize a Repo client](#3--initialize-a-repo-client)
 * [Fetch all the repositories](#4--fetch-all-the-repositories)
 * [Build Android](#5-build-android)
 * [Create micro SD card](#6-create-micro-sd-card)
 * [Boot Android](#7-boot-android)
 * [Use Android](#8-use-android)
 * [Known Issues](#9-issues)
 * [Resources](#10-resources)

Getting Started
---------------
##1.  Configure Development Machine##

***
**Note:**
Android requires a 64-bit build machine.  These instructions have been tested on an Ubuntu 12.04 and 12.10 installation but recent Ubuntu releases should work.  See http://source.android.com/source/initializing.html for more details.
***

Android only likes official Java:

    $ sudo add-apt-repository ppa:webupd8team/java
    $ sudo apt-get update && sudo apt-get install oracle-java6-installer

Grab some other packages:

    $ sudo apt-get install git gnupg flex bison gperf build-essential zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386 uboot-mkimage
    $ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so

[optional] To use ADB (the Android Debug Bridge), allow access to the USB port on Android devices:

    $ sudo sh -c "echo 'SUBSYSTEM==\"usb\", ATTR{idVendor}==\"18d1\", ATTR{idProduct}==\"d002\", MODE=\"0666\"' >> /etc/udev/rules.d/51-android.rules"

##2.  Install Repo.##
***
**Note:**
Users of the [Gumstix Yocto Manifest] (https://github.com/gumstix/Gumstix-YoctoProject-Repo "Gumstix Yocto Manifest") can skip this step; you've already installed repo.
***

Download the Repo script:

    $ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > repo

Make it executable:

    $ chmod a+x repo

Move it on to your system path:

    $ sudo mv repo /usr/local/bin/

If it is correctly installed, you should see a *Usage* message when invoked
with the help flag:

    $ repo --help

##3.  Initialize a Repo client.##

Create an empty directory to hold your working files:

    $ mkdir android
    $ cd android

Tell Repo where to find the manifest for your particular device.

    $ repo init -u git://github.com/gumdroid/manifest.git -b <pepper|overo>

A successful initialization will end with a message stating that Repo is
initialized in your working directory. Your directory should now
contain a *.repo* directory where repo control files such as the manifest are
stored but you should not need to touch this directory.

***
**Note:**
You can use the **-b** switch to specify the branch of the repository to use.

To test out the bleeding edge, type:

    $ repo init -u git://github.com/gumdroid/manifest.git -b dev/<pepper|overo>
    $ repo sync

To get back to the known stable version, type:

    $ repo init -u git://github.com/gumdroid/manifest.git -b <devname>
    $ repo sync

To learn more about repo, look at http://source.android.com/source/version-control.html 
***

##4.  Fetch all the repositories##

    $ repo sync

Now go turn on the coffee machine as this may take an hour or so depending on
your internet connection.

##5. Build Android##

Android includes a pre-built cross-compiler for ARM systems and uses a **make**-based
build system.  Android provides an environment setup script to save a little typing
during development.

    $ source build/envsetup.sh
    $ lunch <pepper|overo>-eng

Now the environment is configured, we can build using the **m** command---an alias for
**make** setup in the previous step.  Try 'm help' to see other options.

    $ m -j8 gumstix

Use the **-j** flag to specify how many threads to use while building. As a general rule,
use twice the number of cores on your development machine.  Now go have a long lunch while
the fan on your computer whirs.

***
**Note:**
There are several components that get built by the _gumstix_ make target which can be built
and cleaned individually during development.  See the files in device/gumstix/common/.

The *systemtarball* target builds a rootfile system tarball that is mounted as a read-only
ext4 filesystem from the second partition of your bootable microSD card. The *userdatatarball*
target builds a tarball that should be uncompressed onto the fourth partition of your microSD
card as a writable area for the OS and can include any application data, photos, and other media.

The build output can be found in the *out* directory; to start fresh you could delete this whole
directory or type 'm clobber'.  After a successful build, you should find a *system.tar.bz2* and
a *userdata.tar.bz2* in *out/target/product/<pepper|overo>*.

We also have targets to build the files needed to boot and run our Gumstix system:

    $ m -j8 build-uboot build-kernel build-sgx

This should build the *mlo* and *u-boot.img* bootloaders from the *uboot* repository as well as
a Linux *uImage* file from the *kernel* repository.  Binaries for the SGX hardware graphical accelerator
are built from the *device/ti/sgx* respository.  The kernel and SGX builds are done in-tree as SGX can't be
built out of tree but the build output is copied to the *out/target/product/<pepper|overo>/boot*
directory.  These targets can be cleaned too: *clean-uboot*, *clean-kernel*, *clean-sgx*. 

SGX includes both kernel modules and userspace libraries so depend both on the kernel and the
systemtarball. If the kernel is updated, remember to rebuild SGX.  If you rebuild SGX, remember
to rebuild the systemtarball to repackage the updated libraries.
***

**6. Make a Bootable SD Card:**

In the build output directory `out/target/product/<pepper|overo>`, you will find these files necessary to build a bootable microSD card.

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

Now that everything is built, we need to format and copy the files over to a microSD card to boot
our Gumstix system.  Insert a blank (or at least, with nothing you want to keep) microSD card of
at least 4GB to your development machine.  Use *dmesg* to [figure out your device name]
(http://gumstix.org/getting-started-guide/242-create-a-bootable-microsd-card.html).

    $ mkandroidsd -d <device-name> -p <pepper|overo>

##7. Boot Android##

Once the card has been created, insert it into your Gumstix system and power on your board!  An animated
Android boot image should appear on the screen followed by the Android lock screen.  The animated screen
will show for several minutes on the first boot as the system configures itself. 

If you configured udev for use with *adb*, hook up a USB cable uto an USB OTG port on your Gumstix
system and follow along with the booting process:

    $ adb logcat 

***
##8. Use Android##

* **Push buttons on Pepper act similarly to the hardware buttons of other Android devices:**

    Switch 2 (S2/GPIO54) is configured as the back button. 
    Switch 3 (S3/GPIO55) is configured as the menu button.

   Additionally, power button can be added to available `ECAP_IN_PWM0_OUT` (Pin 8 in the 40 pin header). 

* **2 Micro USB are configured as the following:**

    USB 1 (J4) is a slave. You can access Android Debug Bridge (ADB) with this.
    USB 2 (J5) is a host. Using a micro USB to USB OTG Host Adapter, you can attach peripherals such as keyboards and mouse. 

* **Fix I2C bus messages errors:**

   There will be peiriodic output to the serial console like this:

        [ 72.078553] omap_i2c omap_i2c.1: timeout waiting for bus ready
        [ 76.758463] omap_i2c omap_i2c.1: controller timed out

   You can fix this by adding 1.5K ohm pullup resistors from `VDD_3V3C` (pin 26) to `I2C0_SCL` (pin 33)  and also to `I2C0_SDA` (pin 31) on the 40 pin header.

* **Install sample media files:**
   You can download a sample media package from [here](https://s3.amazonaws.com/gumstix-misc/android/sample_media/gumstix_media_samples.tar.gz).
   
        $ tar xaf gumstix_media_samples.tar.gz
        $ adb push gumstix_media_samples /data/media/0/

   Then go to `Dev Tools > Media Provider > Scan`. 

##9. Issues##
The following peripherals currently have issues:

###9.1 Marvell mwifiex SD8787 and ethernet###

   Firstly, Marvell's SD8787 driver binary does not support module parameters, but Android assumes it does, for example, in  _/sys/module/wlan/parameters_ to configure operation mode (ie. station, AP, P2P). 
   Specifically, the network bringup fails at the line 906 of [hardware/libhardware_legacy/wifi/wifi.c](https://android.googlesource.com/platform/hardware/libhardware_legacy/+/android-4.3.1_r1/wifi/wifi.c)

	wifi.c:906 fd = TEMP_FAILURE_RETRY(open(WIFI_DRIVER_FW_PATH_PARAM, O_WRONLY));

It cannot open the parameters file descripter: 

	E/WifiHW  (   81): Failed to open wlan fw path param (No such file or directory)

The mwifiex module indeed does not have parameters:

	root@pepper:/sys/module/mwifiex # ls
	uevent
	version

Consequently, RFKill cannot manage the wireless chipset: 
 
	I/wpa_supplicant(  825): rfkill: Cannot open RFKILL control device

   Secondly, Android now uses Broadcom's [Bluedroid](https://source.android.com/devices/bluetooth.html) instead of Bluez. This new bluetooth stack does not support bluetooth devices using UART. Possible workarounds include writing a custom bluetooth profile to work with Bluedroid and swapping back Bluez entirely.  

   Lastly, while ethernet works out of box, Android lacks ethernet configuration tools accessible from the user interface. 

###9.2 TI TLV320AIC3106###

   The audio interface is configured, yet is not reliable. Android Music application play wav and mp3 files, and user's screen touch gets audio feedback, but audio stops with no obvious log output. A good debug point would be the hardware application layer (HAL) for audio which is found in *device/gumstix/pepper/libaudio*. Another possible fail point would be the mixer settings. Although Android uses ALSA just like the Yocto image, it uses *tinymix*. 

##10. Resources ##
* Gumstix Developer Center - http://gumstix.org
* Gumstix Mailing List - https://lists.sourceforge.net/lists/listinfo/gumstix-users
* Pepper board schematic - http://pubs.gumstix.com/boards/PEPPER/R4021/
* Gumstix Android repository - https://github.com/gumdroid/
