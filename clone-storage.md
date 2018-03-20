# Clone the phone's storage memory

## Introduction

Rather than working on the phone directly, it's easier and safer to work on an image of the phone's internal memory. We therefore need to dump all the data so we can backup and restore it.

## Finding the storage place

1. Let's have an interactive shell on the device, using in a terminal (of your computer) `adb shell`. It shall give you a prompt such as `shell@android:/ $`. If it doesn't work, be sure your device has USB debugging turned on, as written in the [requirements](https://gsurrel.github.io/PortingAndroid/#requirements).
1. Now, let's elevate to root, not to be blocked by restrictions set on the user: type `su`. The prompt should have changed to ` root@android:/ #`. It it doesn't work, check if your device is rooted. The phone might ask for permission to get root access, be sure to allow it.
1. We can now list all the disks (*I call flash storage and SD cards disks*) and partitions using `ls -ls /dev/`. There should be a big bunch of text, as follows
    ```console
    shell@android:/ $ su
    root@android:/ # ls -l /dev/block
    brw------- root     root       7,   0 1970-01-01 20:29 loop0
    brw------- root     root       7,   1 1970-01-01 20:29 loop1
    brw------- root     root       7,   2 1970-01-01 20:29 loop2
    brw------- root     root       7,   3 1970-01-01 20:29 loop3
    brw------- root     root       7,   4 1970-01-01 20:29 loop4
    brw------- root     root       7,   5 1970-01-01 20:29 loop5
    brw------- root     root       7,   6 1970-01-01 20:29 loop6
    brw------- root     root       7,   7 1970-01-01 20:29 loop7
    brw------- root     root     179,   0 1970-01-01 20:29 mmcblk0     → First ('zero') real physical disk
    brw------- root     root     179,   1 1970-01-01 20:29 mmcblk0p1   →   Partition 1
    brw------- root     root     179,  10 1970-01-01 20:29 mmcblk0p10  →   Partition 10
    brw------- root     root     179,  11 1970-01-01 20:29 mmcblk0p11  →   Partition 11
    brw------- root     root     179,  12 1970-01-01 20:29 mmcblk0p12  →   Partition 12
    brw------- root     root     179,  13 1970-01-01 20:29 mmcblk0p13  →     ...    
    brw------- root     root     179,  14 1970-01-01 20:29 mmcblk0p14
    brw------- root     root     179,  15 1970-01-01 20:29 mmcblk0p15
    brw------- root     root     179,  16 1970-01-01 20:29 mmcblk0p16
    brw------- root     root     179,  17 1970-01-01 20:29 mmcblk0p17
    brw------- root     root     179,  18 1970-01-01 20:29 mmcblk0p18
    brw------- root     root     179,  19 1970-01-01 20:29 mmcblk0p19
    brw------- root     root     179,   2 1970-01-01 20:29 mmcblk0p2
    brw------- root     root     179,  20 1970-01-01 20:29 mmcblk0p20
    brw------- root     root     179,  21 1970-01-01 20:29 mmcblk0p21
    brw------- root     root     179,  22 1970-01-01 20:29 mmcblk0p22
    brw------- root     root     179,  23 1970-01-01 20:29 mmcblk0p23
    brw------- root     root     179,  24 1970-01-01 20:29 mmcblk0p24
    brw------- root     root     179,  25 1970-01-01 20:29 mmcblk0p25
    brw------- root     root     179,  26 1970-01-01 20:29 mmcblk0p26
    brw------- root     root     179,   3 1970-01-01 20:29 mmcblk0p3
    brw------- root     root     179,   4 1970-01-01 20:29 mmcblk0p4
    brw------- root     root     179,   5 1970-01-01 20:29 mmcblk0p5
    brw------- root     root     179,   6 1970-01-01 20:29 mmcblk0p6
    brw------- root     root     179,   7 1970-01-01 20:29 mmcblk0p7
    brw------- root     root     179,   8 1970-01-01 20:29 mmcblk0p8
    brw------- root     root     179,   9 1970-01-01 20:29 mmcblk0p9
    brw------- root     root     179,  32 1970-01-01 20:29 mmcblk1
    brw------- root     root     179,  33 1970-01-01 20:29 mmcblk1p1
    drwxr-xr-x root     root              1970-01-01 20:29 platform
    brw------- root     root       1,   0 1970-01-01 20:29 ram0
    brw------- root     root       1,   1 1970-01-01 20:29 ram1
    brw------- root     root       1,  10 1970-01-01 20:29 ram10
    brw------- root     root       1,  11 1970-01-01 20:29 ram11
    brw------- root     root       1,  12 1970-01-01 20:29 ram12
    brw------- root     root       1,  13 1970-01-01 20:29 ram13
    brw------- root     root       1,  14 1970-01-01 20:29 ram14
    brw------- root     root       1,  15 1970-01-01 20:29 ram15
    brw------- root     root       1,   2 1970-01-01 20:29 ram2
    brw------- root     root       1,   3 1970-01-01 20:29 ram3
    brw------- root     root       1,   4 1970-01-01 20:29 ram4
    brw------- root     root       1,   5 1970-01-01 20:29 ram5
    brw------- root     root       1,   6 1970-01-01 20:29 ram6
    brw------- root     root       1,   7 1970-01-01 20:29 ram7
    brw------- root     root       1,   8 1970-01-01 20:29 ram8
    brw------- root     root       1,   9 1970-01-01 20:29 ram9
    drwx------ root     root              2018-03-20 16:56 vold
    ```

   The internal memory is in my case `mmcblk0` as it contains a huge number of partitions. The disk `mmcblk1` with a single partition is my SD card.

## Zeroing the free space for good compression

Before doing a bitwise copy of the internal memory, we will fill the empty spaces with zeros, so it can be compressed well. **Be sure** to be connected to the computer using the *MTP protocol*, rather than the *USB mass storage* one. Otherwise, it will not be able to write to the memory shared with the computer. The following commands will do the zeroing:

```console
cd /data
dd if=/dev/zero of=zero.small.file bs=1M count=102400
dd if=/dev/zero of=zero.file bs=100M
sync ; sleep 5 ; sync
rm zero.small.file
rm zero.file

cd /cache
dd if=/dev/zero of=zero.small.file bs=1M count=102400
dd if=/dev/zero of=zero.file bs=100M
sync ; sleep 5 ; sync
rm zero.small.file
rm zero.file

cd /storage/sdcard0/
dd if=/dev/zero of=zero.small.file bs=1M count=102400
dd if=/dev/zero of=zero.file bs=100M
sync ; sleep 5 ; sync
rm zero.small.file
rm zero.file
```

It will take quite some time, please be patient.

## Copying the memory, compressing it, and chunking it

Finally, we can start copying (imaging) the internal memory. The command below will read the disk bytes, compress it on the fly, and chunk it in chunks smaller than 1GB (to avoid problems of creating a file too big for the filesystem) and write it to the external SD card. If you have no SD card available, you can probably stream the output in a SSH tunnel to your computer.

```console
dd if=/dev/block/mmcblk0 bs=96000000 | gzip -c | split -b 1000000000 - /storage/sdcard1/phone.img.gz
```

{% include_relative toc.md %}
