# Running AGL on Intel Minnowboard (and most Intel 64 bits HW)

## Scope

This documentation is aiming at people who want to run Automotive Grade
Linux (AGL) on an Intel Hardware (HW).  
While the reference HW used by AGL project is the Open Source Minnowboard.  
This documentation [MinnowBoard wiki](https://minnowboard.org/) 
can be used to enable most of 64 bits Intel Architecture (IA) using UEFI as boot loader.  
In addition to the MinnowBoard, support for the the [Joule Developer Kit](https://software.intel.com/en-us/iot/hardware/joule/dev-kit) has been added. 
You need to run the 64 bits version of the UEFI boot.  
Minnowbaord Max and Turbo has well as the Joule are both 64 bits capable.

**Note**:

* This page is more focused on please willing to create bespoke AGL image and BSP.

If you are more interested by Apps creation, please visit [ Developing Apps for AGL](<https://wiki.automotivelinux.org/agl-distro/developer>_resources_intel_apps)

UEFI has evolved a lot recently and you likely want to check that your HW firmware is up-to-date, this is mandatory for the Minnowboard and the Joule.

[`https://firmware.intel.com/projects/minnowboard-max`](https://firmware.intel.com/projects/minnowboard-max)  
[`https://software.intel.com/en-us/flashing-the-bios-on-joule`](https://software.intel.com/en-us/flashing-the-bios-on-joule)

## Where to find an AGL bootable image

### Building an AGL image from scratch using Yocto

**Note**:

* An alternative method for building an image is to use the AGL SDK delivered in a Docker container.

There is currently no SDK dedicated to IA but the SDK provided for the Porter Board can build an IA image without changes (just aglsetup.sh needs to call for Intel).

see chapter 2 of [Porter QuickStart](http://iot.bzh/download/public/2016/sdk/AGL-Kickstart-on-Renesas-Porter-board.pdf "wikilink").

#### Download AGL source code

Downloading the AGL sources from the various Git repositories is automated with the repo
tools [to RepoDocumentation](https://source.android.com/source/using-repo.html "wikilink")

To install the repo tool.

```bash
  mkdir -p ~/bin;
  export PATH=~/bin:$PATH;
  curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo;
  chmod a+x ~/bin/repo;
```

#### Configuring for the current *(older)* stable (Charming Chinook 3.0.x)

```bash
  cd AGL-3.0.x;
  repo init -b chinook -m default.xml -u https://gerrit.automotivelinux.org/gerrit/AGL/AGL-repo
```

#### Configuring for master (DD)

```bash
  cd AGL-master;
  repo init -u https://gerrit.automotivelinux.org/gerrit/AGL/AGL-repo;
```

Once that you repo is initialised either with the stable or WIP, you need to sync the repo to fetch the various git trees.

#### Downloading the configured AGL source code

```bash
  repo sync;
```

#### Building the AGL distro

You are now ready to initialise your Yocto build.  
When running the command:

```bash
  source meta-agl/scripts/aglsetup.sh -h
```

You will notice the Intel entries

```bash
  intel-corei7-64
  joule
```

Simply select that entry to replace porter in the -m option.  
**Note:** agl-netboot option is required to create the right initramfs even if you do not boot from a network

```bash
  source meta-agl/scripts/aglsetup.sh \
  -m intel-corei7-64 \
  -b build \
  agl-devel agl-demo agl-appfw-smack agl-netboot
```

**Note:** use the option "-m joule" when building for a Joule developer Kit target.

Start the build **This can take several hours depending of your CPU and
internet connection and will required several GB on /tmp as well as on your build directory**

```bash
  bitbake agl-demo-platform
```

**Your newly baked disk image (.hddimg) will be located at**:  
  `tmp/deploy/images/intel-corei7-64/`

##### Alternative: Download a *ready made* image from AGL web site

The Continuous Integration (CI) process from AGL creates and publish daily and stable builds.  
Pointers to both can be found in [AGL supported HW](https://wiki.automotivelinux.org/agl-distro) (see Reference BSP/Intel).

Once you have validated your process you can start to play/work with the snapshot pointer.

Note that snapshot build may not work.

Follow the directory:  

`intel-corei7-64/deploy/images/intel-corei7-64/`

and download the file:  

`agl-demo-platform-intel-corei7-64.hddimg`

## Create a bootable media

Depending your target HW you will use an USB stick, an SD card or a HDD/SDD.  
The creation process remains the same independently of the selected support.  
It does require to have access to a Linux machine with sudo or root password.

### Insert you removable media in the corresponding interface

### Check the device name where the media can be accessed with the command

```bash
  lsblk
  # Note that you want the name of the raw device not of a partition on the media
  #(eg. /dev/sdc or /dev/mmcblk0)
```

### Download the script mkefi-agl.sh

This script is present in the directory meta-agl/scripts from blowfish 2.0.4, alternatively you can download it from the following Git repo:  

[https://github.com/dominig/mkefi-agl.sh](https://github.com/dominig/mkefi-agl.sh)

### check the available option

```bash
  sh mkefi-agl.sh -v;
```

### create your media with the command ajusted to your configuration

```bash
  sudo sh mkefi-agl.sh MyAglImage.hdd /dev/sdX
  #/dev/sdX is common for USB stick, /dev/mmcblk0 for laptop integrated SD card reader
```

## Boot the image on the target device

1. Insert the created media with the AGL image in the target device

1. Power on the device

1. Select Change one off boot option (generally F12 key during power up)

1. Select your removable device

1. Let AGL boot

**Note:**:

* Depending on the speed of the removable media, the first boot may not complete, in that case simply reboot the device.

This is quite common with USB2 sticks.

By default the serial console is configured and activated at the rate of 115200 bps.

## How to create your 1st AGL application

[Developing Apps for AGL](https://wiki.automotivelinux.org/agl-distro/developer_resources_intel_apps)
