Build Android for Gumstix Boards
================================
This repository provides Repo manifests to build Android for Gumstix products.

***
**Note:**
If you already have an Android build setup and just want the device
directories, you can find them here:

 * **Pepper**: git://github.com/ashcharles/pepper.git
 * **Overo**: git://github.com/ashcharles/overo.git
***

Repo is a tool that enables the management of many git repositories given a 
single *manifest* file.  Tell repo to fetch a manifest from this repository and
it will fetch the git repositories specified in the manifest and, by doing so,
setup an Android build environment for you.

Getting Started
---------------
**1.  Configure Development machine.**

**Note:**
Android requires a 64-bit build machine.  These instructions have been tested on an Ubuntu 12.10 installation but recent Ubuntu releases should work.  See http://source.android.com/source/initializing.html for more details.

Android only likes official Java.

    $ sudo add-apt-repository ppa:webupd8team/java
    $ sudo apt-get update && sudo apt-get install oracle-java6-installer

Grab some other packages.

    $ sudo apt-get install git gnupg flex bison gperf build-essential zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386 uboot-mkimage
    $ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so

** 2. [Optional] To use ADB (the Android Debug Bridge), allow access to the USB port on Android devices.

  $ sudo sh -c "echo 'SUBSYSTEM==\"usb\", ATTR{idVendor}==\"18d1\", ATTR{idProduct}==\"d002\", MODE=\"0666\"' >> /etc/udev/rules.d/51-android.rules"

**3.  Install Repo.**

**Note:**
Users of the [Gumstix Yocto Manifest] (https://github.com/gumstix/Gumstix-YoctoProject-Repo "Gumstix Yocto Manifest") can skip this step; you've already installed repo.

Download the Repo script:

    $ curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > repo

Make it executable:

    $ chmod a+x repo

Move it on to your system path:

    $ sudo mv repo /usr/local/bin/

If it is correctly installed, you should see a *Usage* message when invoked
with the help flag.

    $ repo --help

**3.  Initialize a Repo client.**

Create an empty directory to hold your working files:

    $ mkdir android
    $ cd android

Tell Repo where to find the manifest:

    $ repo init -u git://github.com/ashcharles/manifest.git -b gumdroid-4.3_r3.1


A successful initialization will end with a message stating that Repo is
initialized in your working directory. Your directory should now
contain a *.repo* directory where repo control files such as the manifest are
stored but you should not need to touch this directory.

***
**Note:**
You can use the **-b** switch to specify the branch of the repository to use.

To test out the bleeding edge, type:

    $ repo init -u git://github.com/ashcharles/manifest.git -b dev
    $ repo sync

To get back to the known stable version, type:

    $ repo init -u git://github.com/ashcharles/manifest.git -b master
    $ repo sync

To learn more about repo, look at http://source.android.com/source/version-control.html 
***

**4.  Fetch all the repositories:**

    $ repo sync

Now go turn on the coffee machine as this may take an hour or so depending on
your internet connection.

**5. Build Android:**

Android includes a pre-built cross-compiler for ARM systems and uses a **make**-based
build system.

    $ make TARGET_PRODUCT=<pepper|overo> -j8 systemtarball userdatatarball

Use the **-j** flag to specify how many threads to use while building. As a general rule,
use twice the number of cores on your development machine; I've got a four-core machine.

The *systemtarball* target builds a rootfile system tarball that is mounted as a read-only
ext4 filesystem from the second partition of your bootable microSD card. The *userdatatarball*
target builds a tarball that should be uncompressed onto the fourth partition of your microSD
card as a writable area for the OS and can include any application data, photos, and other media.

The build output can be found in the *out* directory; to start fresh you could delete this whole
directory.  After a successful build, you should find a *system.tar.bz2* and a *userdata.tar.bz2*
in *out/target/product/<pepper|overo>*

We also need to build the files needed to boot and run our Gumstix system:

    $ make TARGET_PRODUCT=<pepper|overo> -j8 bsp

This should build the *mlo* and *u-boot.img* bootloaders from the *uboot* repository as well as
a Linux *uImage* file from the *kernel* repository.  Binaries for the SGX hardware graphical accelerator
are built from the *device/ti/sgx* respository.  All these builds are done in-tree as SGX can't be
built out of tree but the build output is copies to the *out/target/product/<pepper|overo>/boot*
directory.  See the *Makefile* in the top-directory for more details.

To include the freshly created SGX libraries in our image, we need to repackage the systemtarball
(don't worry, it is quick):

    $ make TARGET_PRODUCT=<pepper|overo> -j8 systemtarball

**6. Make a Bootable SD Card:**
Now that everything is built, we need to format and copy the files over to a microSD card to boot
our Gumstix system.  Insert a blank (or at least, with nothing you want to keep) microSD card of
at least 4GB to your development machine.  Use *dmesg* to [figure out your device name]
(http://gumstix.org/getting-started-guide/242-create-a-bootable-microsd-card.html).

    $ out/host/linux-x86/bin/mkcard -d <device-name> -p <pepper|overo>

For example:

    $ out/host/linux-x86/bin/mkcard -d /dev/mmcblk0 -p <pepper|overo>
