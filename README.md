Build Android for Gumstix Boards
================================

This repository provides Repo manifests to build Android for Gumstix products.

***
**Note:**
If you already have an Android build setup and just want the device
directories, you can find them here:

**As of Auguest 2015, the following information only pertains to Gumstix Pepper 43C and Overo . Stay tuned for updates on Duovero.**

 * **Pepper43C**: git://github.com/gumdroid/pepper.git
 * **Overo**: git://github.com/gumdroid/overo.git

***

Repo is a tool that enables the management of many git repositories given a 
single *manifest* file.  Tell repo to fetch a manifest from this repository and
it will fetch the git repositories specified in the manifest and, by doing so,
setup an Android build environment for you.

##Contents##

 * [Configure Development Machine](#1-configure-development-machine)
 * [Install Repo](#2-install-repo)
 * [Initialize a Repo client](#3-initialize-a-repo-client)
 * [Fetch all the repositories](#4-fetch-all-the-repositories)
 * [Build Android](#5-build-android)
 * [Create micro SD card](#6-make-a-bootable-sd-card)
 * [Boot Android](#7-boot-android)
 * [Use Android](#8-use-android)
 * [Known Issues](#9-issues)
 * [Resources](#10-resources)

Getting Started
---------------
##1. Configure Development Machine##

***
**Note:**
Android requires a 64-bit build machine.  These instructions have been tested on an Ubuntu 12.04 and 12.10 installation but recent Ubuntu releases should work.  See http://source.android.com/source/initializing.html for more details.
***

Android only likes official Java:

    $ sudo add-apt-repository ppa:webupd8team/java
    $ sudo apt-get update && sudo apt-get install oracle-java6-installer

Grab some other packages:

    $ sudo apt-get install git gnupg flex bison gperf build-essential zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386 u-boot-tools libswitch-perl
    $ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so

[optional] To use ADB (the Android Debug Bridge), allow Android devices to register with your development machine:

    $ sudo sh -c "echo 'SUBSYSTEM==\"usb\", ATTR{idVendor}==\"18d1\", ATTR{idProduct}==\"d002\", MODE=\"0666\"' >> /etc/udev/rules.d/51-android.rules"

##2. Install Repo##
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

##3. Initialize a Repo Client##

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

    $ repo init -u git://github.com/gumdroid/manifest.git -b <pepper|overo>
    $ repo sync

To learn more about repo, look at http://source.android.com/source/version-control.html 
***

##4. Fetch All the Repositories##

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
**make** setup in the previous step. Use the **-j** flag to specify how many threads to
use while building. As a general rule, use twice the number of cores on your development machine.

    $ m -j8 gumstix

Try `hmm` to see other options. Now go have a long lunch while the fan on your computer whirs.

***
**Note:**
There are several components that get built by the **gumstix** make target which can be built
and cleaned individually during development.  See the files in `device/gumstix/common/` for details.

The **systemtarball** target builds a rootfile system tarball that is mounted as a read-only
ext4 filesystem from the second partition of your bootable microSD card. The **userdatatarball**
target builds a tarball that should be uncompressed onto the fourth partition of your microSD
card as a writable area for the OS and can include any application data, photos, and other media.

The build output can be found in the *out* directory; to start fresh you could delete this whole
directory or type `m clobber`.  After a successful build, you should find a **system.tar.bz2** and
a **userdata.tar.bz2** in `out/target/product/<pepper|overo>`.

We also have targets to build the files needed to boot and run our Gumstix system:

    $ m -j8 uboot linux sgx

This should build the **MLO** and **u-boot.img** bootloaders from the *uboot* repository as well as
a Linux image from the *kernel* repository.  Binaries for the SGX hardware graphical accelerator
are built from the *device/ti/sgx* respository.  The kernel and SGX builds are done in-tree as SGX can't be
built out of tree but the build output is copied to the `out/target/product/<pepper|overo>/boot`
directory.  These targets can be cleaned too: **clean-uboot**, **clean-kernel**, **clean-sgx**. 

SGX includes both kernel modules and userspace libraries so depend both on the kernel and the
systemtarball. If the kernel is updated, remember to rebuild SGX.  If you rebuild SGX, remember
to rebuild the systemtarball to repackage the updated libraries.
***

##6. Make a Bootable SD Card##

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
at least 4GB to your development machine.  Use **dmesg** to [figure out your device name]
(http://gumstix.org/getting-started-guide/242-create-a-bootable-microsd-card.html).

    $ mkandroidsd /dev/mmcblk0 <pepper|overo>

You may be prompted for as sudo password when needed.

##7. Boot Android##

Once the card has been created, insert it into your Gumstix system and power on your board!  An animated
Android boot image should appear on the screen followed by the Android lock screen.  The animated screen
will show for several minutes on the first boot as the system configures itself. 

If you configured your system for use with **adb**, hook up a USB cable to a USB OTG port on your Gumstix
system and follow along with the booting process:

    $ adb logcat 

***
##8. Use Android##

* **Push buttons on Pepper43C act similarly to the hardware buttons of other Android devices:**

    Switch 2 (GPIO54) is configured as the Back button.
    GPIO55 (marked '55' on the 'J8' 20-pin header) is configured as the Menu button.
    GPIO7 (marked '07' on the 'J8' 20-pin header) is configured as the Power button.

* **2 Micro USB are configured as the following:**

    USB 1 (J6) is a slave. You can access Android Debug Bridge (ADB) with this.
    USB 2 (J7) is an USB-UART serial console.

##9. Issues##

Please refer to [Issues](https://github.com/gumdroid/manifest/issues?state=open) page

##10. Resources ##
* Gumstix Developer Center - http://gumstix.org
* Gumstix Mailing List - https://lists.sourceforge.net/lists/listinfo/gumstix-users
* Gumstix Android repository - https://github.com/gumdroid/
* Useful tips on the Android make system - http://www.lindusembedded.com/blog/2010/12/10/the-android-build-system/
* Android Source Reference - http://source.android.com
