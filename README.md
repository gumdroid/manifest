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

Getting Started
---------------
**1.  Configure Development machine.**

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

**2.  Install Repo.**

Download the Repo script:

    $ curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > repo

Make it executable:

    $ chmod a+x repo

Move it on to your system path:

    $ sudo mv repo /usr/local/bin/

If it is correctly installed, you should see a Usage message when invoked
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
contain a .repo directory where repo control files such as the manifest are
stored but you should not need to touch this directory.

***
**Note:**
You can use the **-b** switch to specify the branch of the repository to use.

The **-m** switch selects the manifest file (default is *default.xml*).

To test out the bleeding edge, type:

    $ repo init -u git://github.com/ash/manifest.git -b dev
    $ repo sync

To get back to the known stable version, type:

    $ repo init -u git://github.com/ash/manifest.git -b master
    $ repo sync

To learn more about repo, look at http://source.android.com/source/version-control.html 
***

**4.  Fetch all the repositories:**

    $ repo sync

Now go turn on the coffee machine as this may take an hour or so depending on
your connection.

**5. Build Android:**

    $ make TARGET_PRODUCT=pepper -j8 bsp systemtarball boottarball userdatatarball


