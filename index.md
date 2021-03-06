# Home

## Introduction

The mail goal of this website is to document how to port [Android Lineage OS](https://lineageos.org/) to a specific device ([see below](#target-device). At the time of this writing, the only thing avaiable is **root access**.

I envision to try to port [TWRP](https://twrp.me/) first, because it is most probably faster to get it working, plus it gives useful options to backup and restore the system.

## Target device

The target is a [Huawei G740-L00](https://www.gsmarena.com/huawei_ascend_g740-5840.php), a cheap phone running Android 4.1.2 thanks to a dual-core 1.2GHz CPU with the help of 1GB of RAM. Nothing amazing, except if it can runs Android 8 Oreo. This could eventually be possible as now Android can have a *Go* declination targeting low-end phones, so it *might* run.

## Requirements

The work will be done using a **Linux computer**, assuming **root access** to the phone with an **unlocked bootloader**, which has been factory reset. The [USB debugging must be enabled](https://developer.android.com/studio/debug/dev-options.html) and the [ADB tools](https://developer.android.com/studio/run/oem-usb.html) installed. To be sure to have to same command-line utilities, installing a [terminal emulator](https://f-droid.org/packages/jackpal.androidterm) and a [BusyBox package](https://f-droid.org/packages/ru.meefik.busybox) can't harm.

*Note: I favor FLOSS software, hence linking to F-Droid. It has the side benefit of not hiding the APKs, so you can directly download and install them with `adb install file.apk`*

## Documentation

From early investigation, I think that [Jonathan Levin](http://newandroidbook.com/)'s [Android Internals::A Confectioner's Cookbook - Power User's View](http://newandroidbook.com/AIvI-M-RL1.pdf) will be very handy, along with [**xda**developers forums](https://forum.xda-developers.com/).

{% include_relative toc.md %}
