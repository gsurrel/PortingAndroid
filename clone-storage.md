# Clone the phone's storage memory

## Introduction

Rather than working on the phone directly, it's easier and safer to work on an image of the phone's internal memory. We therefore need to dump all the data so we can backup and restore it.

## Finding the storage place

1. Let's have an interactive shell on the device, using in a terminal (of your computer) `adb shell`. It shall give you a prompt such as `shell@android:/ $`. If it doesn't work, be sure your device has USB debugging turned on, as written in the [requirements](https://gsurrel.github.io/PortingAndroid/#requirements).
1. Now, let's elevate to root, not to be blocked by restrictions set on the user: type `su`. The prompt should have changed to `root@android:/ #`. It it doesn't work, check if your device is rooted. The phone might ask for permission to get root access, be sure to allow it.
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
    brw------- root     root     179,   0 1970-01-01 20:29 mmcblk0     → First disk (internal)
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
    brw------- root     root     179,  32 1970-01-01 20:29 mmcblk1     → Second disk (SD card)
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

```
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

It will take quite some time, please be patient. Seeing `zero.file: write error: No space left on device` is what we are looking for: it is a file full of zeros, filling up the whole avaiable memory. It is removed just after to free-up space.

**Note:** I chose to create a big file of zeros in the the `/data`, `/cache` and `/storage` directories as they are the mount points of the biggest partitions. Little deduction helps you guess that multiple mount-points share the same physical space on disk as they have exactly the same total size:

```console
root@android:/ # df
Filesystem             Size   Used   Free   Blksize
/dev                   403M   136K   403M   4096
/mnt/asec              403M     0K   403M   4096
/mnt/obb               403M     0K   403M   4096
/system               1009M   603M   405M   4096
/data                    1G   113M   998M   4096
/cache                 189M     4M   184M   4096
/persist                 7M     4M     3M   4096
/firmware               63M    44M    19M   16384
/cust                  252M    52M   199M   4096
/storage/sdcard0         4G     2G     2G   32768
/storage/sdcard1        14G     5G     9G   32768
/mnt/secure/asec        14G     5G     9G   32768
```

## Copying the whole memory, compressing it, and chunking it

Finally, we can start copying (imaging) the internal memory. It's possible to save the disk image either on a SD card if the phone has a SD card slot, or directly transfer it through ADB.

### With a SD card available

The command below will read the disk bytes, compress it on the fly, and chunk it in chunks smaller than 1GB (to avoid problems of creating a file too big for the filesystem if it's FAT32) and write it to the external SD card.

```
dd if=/dev/block/mmcblk0 bs=96000000 | gzip -c | split -b 1000000000 - /storage/sdcard1/phone.img.gz
```

This step is also quite long. If it is too long, wait faster!

### Without a SD card

The command is quite similar: read the memory, compress it. This writes the compressed output to the standard output, so we can redirect it to a local file. The `pv` par is optionnal, it's only for having feedback on the progress of the command.

```
adb shell "cat /dev/block/sda | gzip -" | pv > ../phone.img.gz
```

This step is not faster than using the SD card, so same rule apply: if it is too long, wait faster!

## Copying partition by partition, compressing it, and chunking it

We have a big backup which contains everything, including the partition table. It is nice (I guess) but not the most convinient. We can also image every partition individually, with their individual names (the `msm_sdcc.1` might change across devices and Android versions):

```console
root@android:/ # ls -l /dev/block/platform/msm_sdcc.1/by-name/
lrwxrwxrwx root     root              1970-01-01 20:29 aboot -> /dev/block/mmcblk0p6
lrwxrwxrwx root     root              1970-01-01 20:29 aboot2 -> /dev/block/mmcblk0p7
lrwxrwxrwx root     root              1970-01-01 20:29 boot -> /dev/block/mmcblk0p21
lrwxrwxrwx root     root              1970-01-01 20:29 cache -> /dev/block/mmcblk0p23
lrwxrwxrwx root     root              1970-01-01 20:29 cust -> /dev/block/mmcblk0p22
lrwxrwxrwx root     root              1970-01-01 20:29 fsg -> /dev/block/mmcblk0p9
lrwxrwxrwx root     root              1970-01-01 20:29 grow -> /dev/block/mmcblk0p26
lrwxrwxrwx root     root              1970-01-01 20:29 misc -> /dev/block/mmcblk0p16
lrwxrwxrwx root     root              1970-01-01 20:29 modem -> /dev/block/mmcblk0p13
lrwxrwxrwx root     root              1970-01-01 20:29 modemst1 -> /dev/block/mmcblk0p10
lrwxrwxrwx root     root              1970-01-01 20:29 modemst2 -> /dev/block/mmcblk0p11
lrwxrwxrwx root     root              1970-01-01 20:29 oeminfo -> /dev/block/mmcblk0p12
lrwxrwxrwx root     root              1970-01-01 20:29 pad -> /dev/block/mmcblk0p20
lrwxrwxrwx root     root              1970-01-01 20:29 persist -> /dev/block/mmcblk0p18
lrwxrwxrwx root     root              1970-01-01 20:29 recovery -> /dev/block/mmcblk0p14
lrwxrwxrwx root     root              1970-01-01 20:29 recovery2 -> /dev/block/mmcblk0p15
lrwxrwxrwx root     root              1970-01-01 20:29 rpm -> /dev/block/mmcblk0p4
lrwxrwxrwx root     root              1970-01-01 20:29 sbl1 -> /dev/block/mmcblk0p1
lrwxrwxrwx root     root              1970-01-01 20:29 sbl2 -> /dev/block/mmcblk0p2
lrwxrwxrwx root     root              1970-01-01 20:29 sbl3 -> /dev/block/mmcblk0p3
lrwxrwxrwx root     root              1970-01-01 20:29 splash -> /dev/block/mmcblk0p17
lrwxrwxrwx root     root              1970-01-01 20:29 ssd -> /dev/block/mmcblk0p8
lrwxrwxrwx root     root              1970-01-01 20:29 system -> /dev/block/mmcblk0p24
lrwxrwxrwx root     root              1970-01-01 20:29 tombstones -> /dev/block/mmcblk0p19
lrwxrwxrwx root     root              1970-01-01 20:29 tz -> /dev/block/mmcblk0p5
lrwxrwxrwx root     root              1970-01-01 20:29 userdata -> /dev/block/mmcblk0p25
```

We can using the following simple shell scripting to do something similar as previously: we save each partition with its name in a directory at the root of the sdcard named `phone_partitions`. I removed the chunking as the biggest partition is small enough for the target filesystem.

```console
root@android:/ # mkdir /storage/sdcard1/phone_partitions/
root@android:/ # cd /dev/block/platform/msm_sdcc.1/by-name/
root@android:/dev/block/platform/msm_sdcc.1/by-name/ # for f in *; do dd if=$f bs=96000000 | gzip -c > /storage/sdcard1/phone_partitions/$f.img.gz; done
```

## More information about partitions

To know more about the partitions, check [*El Grande Partition Table Reference* in the **xda**developer forums](https://forum.xda-developers.com/showthread.php?t=1959445). I save the output here as I don't know if it will be useful in the future. Below is the `fdisk` output, ran on both the device and partitions (to get more info therwise fails). It has been restyled by hand to be a little more legible.

```console
root@android:/ # fdisk -l /dev/block/mmcblk*

% Disk /dev/block/mmcblk0: 7456 MB, 7818182656 bytes, 15269888 sectors
946 cylinders, 256 heads, 63 sectors/track
Units: cylinders of 16128 * 512 = 8257536 bytes

Device             Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
/dev/block/mmcblk0p1    0,0,1       1023,255,63          1 4294967295 4294967295 2047G ee EFI GPT
Partition 1 has different physical/logical start (non-Linux?):
     phys=(0,0,1) logical=(0,0,2)
Partition 1 has different physical/logical end:
     phys=(1023,255,63) logical=(266305,4,4)

% Disk /dev/block/mmcblk0p1: 0 MB, 262144 bytes, 512 sectors
8 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p1 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p10: 4 MB, 4194304 bytes, 8192 sectors
128 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p10 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p11: 4 MB, 4194304 bytes, 8192 sectors
128 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p11 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p12: 32 MB, 33554432 bytes, 65536 sectors
1024 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p12 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p13: 64 MB, 67108864 bytes, 131072 sectors
2048 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Device                Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
/dev/block/mmcblk0p13p1 6f 357,116,40  357,32,45    778135908 1919645538 1141509631  544G 72 Unknown
Partition 1 does not end on cylinder boundary
/dev/block/mmcblk0p13p2 69 288,115,43  367,114,50   168689522 2104717761 1936028240  923G 65 Unknown
Partition 2 does not end on cylinder boundary
/dev/block/mmcblk0p13p3 73 366,32,33   357,32,43   1869881465 3805909656 1936028192  923G 79 Unknown
Partition 3 does not end on cylinder boundary
/dev/block/mmcblk0p13p4 74 372,97,50   0,10,0               0 3637226495 3637226496 1734G  d Unknown
Partition 4 does not end on cylinder boundary

Partition table entries are not in disk order

% Disk /dev/block/mmcblk0p14: 16 MB, 16777216 bytes, 32768 sectors
512 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p14 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p15: 16 MB, 16777216 bytes, 32768 sectors
512 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p15 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p16: 4 MB, 4194304 bytes, 8192 sectors
128 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p16 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p17: 4 MB, 4194304 bytes, 8192 sectors
128 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p17 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p18: 8 MB, 8388608 bytes, 16384 sectors
256 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p18 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p19: 8 MB, 8388608 bytes, 16384 sectors
256 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p19 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p2: 0 MB, 262144 bytes, 512 sectors
8 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p2 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p20: 0 MB, 655360 bytes, 1280 sectors
20 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p20 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p21: 12 MB, 12582912 bytes, 24576 sectors
384 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p21 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p22: 256 MB, 268435456 bytes, 524288 sectors
8192 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p22 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p23: 192 MB, 201326592 bytes, 393216 sectors
6144 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p23 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p24: 1024 MB, 1073741824 bytes, 2097152 sectors
32768 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p24 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p25: 1128 MB, 1182793728 bytes, 2310144 sectors
36096 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p25 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p26: 4667 MB, 4894735872 bytes, 9560031 sectors
149375 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Device                Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type

% Disk /dev/block/mmcblk0p3: 0 MB, 524288 bytes, 1024 sectors
16 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p3 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p4: 0 MB, 262144 bytes, 512 sectors
8 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p4 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p5: 1 MB, 1048576 bytes, 2048 sectors
32 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p5 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p6: 2 MB, 2097152 bytes, 4096 sectors
64 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p6 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p7: 2 MB, 2097152 bytes, 4096 sectors
64 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p7 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p8: 0 MB, 8192 bytes, 16 sectors
0 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p8 doesn't contain a valid partition table

% Disk /dev/block/mmcblk0p9: 4 MB, 4194304 bytes, 8192 sectors
128 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes
Disk /dev/block/mmcblk0p9 doesn't contain a valid partition table
```

For information, here is another source for finding the partitions caracteristics:
```console
root@android:/ # cat /proc/partitions
major minor  #blocks  name

 179        0    7634944 mmcblk0
 179        1        256 mmcblk0p1
 179        2        256 mmcblk0p2
 179        3        512 mmcblk0p3
 179        4        256 mmcblk0p4
 179        5       1024 mmcblk0p5
 179        6       2048 mmcblk0p6
 179        7       2048 mmcblk0p7
 179        8          8 mmcblk0p8
 179        9       4096 mmcblk0p9
 179       10       4096 mmcblk0p10
 179       11       4096 mmcblk0p11
 179       12      32768 mmcblk0p12
 179       13      65536 mmcblk0p13
 179       14      16384 mmcblk0p14
 179       15      16384 mmcblk0p15
 179       16       4096 mmcblk0p16
 179       17       4096 mmcblk0p17
 179       18       8192 mmcblk0p18
 179       19       8192 mmcblk0p19
 179       20        640 mmcblk0p20
 179       21      12288 mmcblk0p21
 179       22     262144 mmcblk0p22
 179       23     196608 mmcblk0p23
 179       24    1048576 mmcblk0p24
 179       25    1155072 mmcblk0p25
 179       26    4780015 mmcblk0p26
 179       32   15558144 mmcblk1
 179       33   15557120 mmcblk1p1

root@android:/ # cat /proc/mounts
rootfs / rootfs ro,relatime 0 0
tmpfs /dev tmpfs rw,nosuid,relatime,mode=755 0 0
devpts /dev/pts devpts rw,relatime,mode=600 0 0
proc /proc proc rw,relatime 0 0
sysfs /sys sysfs rw,relatime 0 0
none /acct cgroup rw,relatime,cpuacct 0 0
tmpfs /mnt/asec tmpfs rw,relatime,mode=755,gid=1000 0 0
tmpfs /mnt/obb tmpfs rw,relatime,mode=755,gid=1000 0 0
none /dev/cpuctl cgroup rw,relatime,cpu 0 0
/dev/block/platform/msm_sdcc.1/by-name/system /system ext4 ro,relatime,data=ordered 0 0
/dev/block/platform/msm_sdcc.1/by-name/userdata /data ext4 rw,nosuid,nodev,relatime,noauto_da_alloc,data=ordered 0 0
/dev/block/platform/msm_sdcc.1/by-name/cache /cache ext4 rw,nosuid,nodev,relatime,data=ordered 0 0
/dev/block/platform/msm_sdcc.1/by-name/persist /persist ext4 rw,nosuid,nodev,relatime,data=ordered 0 0
/dev/block/platform/msm_sdcc.1/by-name/modem /firmware vfat ro,relatime,uid=1000,gid=1000,fmask=0337,dmask=0227,codepage=cp437,iocharset=iso8
859-1,shortname=lower,errors=remount-ro 0 0
/dev/block/platform/msm_sdcc.1/by-name/cust /cust ext4 ro,relatime,data=ordered 0 0
/dev/block/vold/179:26 /storage/sdcard0 vfat rw,nosuid,nodev,noexec,relatime,uid=1000,gid=1015,fmask=0002,dmask=0002,allow_utime=0020,codepag
e=cp437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remount-ro 0 0
/dev/block/vold/179:33 /storage/sdcard1 vfat rw,nosuid,nodev,noexec,relatime,uid=1000,gid=1015,fmask=0002,dmask=0002,allow_utime=0020,codepag
e=cp437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remount-ro 0 0
/dev/block/vold/179:33 /mnt/secure/asec vfat rw,nosuid,nodev,noexec,relatime,uid=1000,gid=1015,fmask=0002,dmask=0002,allow_utime=0020,codepag
e=cp437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remount-ro 0 0
tmpfs /storage/sdcard1/.android_secure tmpfs ro,relatime,size=0k,mode=000 0 0
```

{% include_relative toc.md %}
