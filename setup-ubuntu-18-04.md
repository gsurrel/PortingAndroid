# Setup Ubuntu 18.04

This page explains the steps to have a working installation of Ubuntu 18.04. I will be running everything in a VirtualBox virtual machine to  have the cleanest environment with few differences with persons following this guide. Normal system updates should always be installed, it will not be mentionned here.

## Install the Android platform tools

In a terminal, use the following command to install `adb` and `fastboot`, along with other tools and their dependencies:

```console
sudo apt install android-sdk-platform-tools
```
Using `fdisk` gives informations about partitions. Let's install it:

```console
sudo apt install fdisk
```

