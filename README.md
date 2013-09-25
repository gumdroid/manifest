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
**1.  Install Repo.**

Download the Repo script:

    $ curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > repo

Make it executable:

    $ chmod a+x repo

Move it on to your system path:

    $ sudo mv repo /usr/local/bin/

If it is correctly installed, you should see a Usage message when invoked
with the help flag.

    $ repo --help

**2.  Initialize a Repo client.**

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

**3.  Fetch all the repositories:**

    $ repo sync

Now go turn on the coffee machine as this may take an hour or so depending on
your connection.

**4. Build Android:**

    $ make TARGET_PRODUCT=pepper -j8 bsp systemtarball boottarball userdatatarball


