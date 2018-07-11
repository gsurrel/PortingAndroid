# Clone the phone's storage memory

## Introduction

Rather than working on the phone directly, it's easier and safer to work on an image of the phone's internal memory. We therefore need to dump all the data so we can backup and restore it.

## Finding the storage place

1. Let's have an interactive shell on the device, using in a terminal (of your computer) `adb shell`. It shall give you a prompt such as `shell@android:/ $`. If it doesn't work, be sure your device has USB debugging turned on, as written in the [requirements](https://gsurrel.github.io/PortingAndroid/#requirements).
1. Now, let's elevate to root, not to be blocked by restrictions set on the user: type `su`. The prompt should have changed to `root@android:/ #`. It it doesn't work, check if your device is rooted. The phone might ask for permission to get root access, be sure to allow it./
1. We can now list all the disks (*I call flash storage and SD cards disks*) and partitions using `ls -ls /dev/block/`. There should be a big bunch of text, as follows:
    ```console
    shell@android:/ $ su
    root@android:/ # ls -l /dev/block/
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

## Copying the WHOLE MEMORY, compressing it, and chunking it

Finally, we can start copying (imaging) the internal memory. It's possible to save the disk image either on a SD card if the phone has a SD card slot, or directly transfer it through ADB.

### With a SD card

The command below will read the disk bytes, compress it on the fly, and chunk it in chunks smaller than 1GB (to avoid problems of creating a file too big for the filesystem if it's FAT32) and write it to the external SD card.

```
dd if=/dev/block/mmcblk0 bs=96000000 | gzip -c | split -b 1000000000 - /storage/sdcard1/phone.img.gz
```

This step is also quite long. If it is too long, wait faster!

### Without a SD card

The command is quite similar: read the memory, compress it. This writes the compressed output to the standard output, so we can redirect it to a local file. The `pv` par is optionnal, it's only for having feedback on the progress of the command.

```
adb forward tcp:7080 tcp:8080; adb shell "cat /dev/block/sda | gzip -c 2>/dev/null | nc -l -p 8080"& sleep 1; nc localhost 7080 > phone.img.gz; adb forward --remove tcp:7080
```

This step is similarly fast than using the SD card, so same rule apply: if it is too long, wait faster!

## Copying PARTITION BY PARTITION, compressing it, and chunking it

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

### With a SD card

We can using the following simple shell scripting to do something similar as previously: we save each partition with its name in a directory at the root of the sdcard named `phone_partitions`. I removed the chunking as the biggest partition is small enough for the target filesystem.

```console
root@android:/ # mkdir /storage/sdcard1/phone_partitions/
root@android:/ # cd /dev/block/platform/msm_sdcc.1/by-name/
root@android:/dev/block/platform/msm_sdcc.1/by-name/ # for f in *; do dd if=$f bs=96000000 | gzip -c > /storage/sdcard1/phone_partitions/$f.img.gz; done
```

### Without a SD card

Here is a version without requiring the SD card, transferring the data directly through ADB in the current folder in compressed archives:

```console
$ adb forward tcp:7080 tcp:8080; for f in $(adb shell "ls /dev/block/platform/msm_sdcc.1/by-name/"); do f=$(echo $f | tr -d "\n" | tr -d "\r"); adb shell "cd /data/local/tmp; mkfifo $f.img.gz; su -c dd if=/dev/block/platform/msm_sdcc.1/by-name/$f bs=96000000 | gzip -c 2>/dev/null | tee $f.img.gz | nc -l -p 8080 & md5sum $f.img.gz >> sums.md5; rm $f.img.gz;"& sleep 1; echo -e "nc localhost 7080 > $f.img.gz << EOF\nEOF\n" | sh; wait $(jobs -rp); done; adb forward --remove tcp:7080; adb pull /data/local/tmp/sums.md5; adb shell "rm /data/local/tmp/sums.md5"; md5sum -c sums.md5
```
The previous massive command does the following:

1. Open the USB connection as a network connection (LAN) `adb forward tcp:7080 tcp:8080;`
1. For each partition in the phone: `for f in $(adb shell "ls /dev/block/platform/msm_sdcc.1/by-name/"); do`
    1. Clean the partition name `f` of line-returns: `f=$(echo $f | tr -d "\n" | tr -d "\r");`
    1. Run a command on the phone which: `adb shell "…";`
        1. Move to the temp directory `cd /data/local/tmp;`
        1. Creates a fifo called `f`.img.gz `mkfifo $f.img.gz;`
        1. Reads the raw partition `f` memory `su -c dd if=/dev/block/platform/msm_sdcc.1/by-name/$f bs=96000000`
        1. Compresses it (while hiding error messages) `| gzip -c 2>/dev/null`
        1. Duplicates the output to the fifo (*file*) `f`.img.gz ` | tee $f.img.gz`
        1. Sends the gzipped result over the network-over-USB connection `| nc -l -p 8080`
        1. And in parallel, computes locally the hash of the gzipped result reading from the fifo `& md5sum $f.img.gz`
        1. Appends it to a file called sums.md5 `>> sums.md5;`
        1. And removes the fifo `rm $f.img.gz;`
    1. In parallel on the computer: `&`
        1. Waits for a second to give time to the phone `sleep 1;`
        1. Creates a command string which: `echo -e "…"`
            - Starts the process to receive the file over the network-over-USB `nc localhost 7080 > $f.img.gz`
            - Simulates the user to press the Enter key to close the connection `<< EOF\nEOF\n`
        1. Command string which is executed immediately `| sh;`
        1. Waits until the phone finishes `wait $(jobs -rp);`
1. Ends the loop `done;`
1. Removes the network-over-USB redirection `adb forward --remove tcp:7080`
1. Retrieve the md5 sums from the phone `adb pull /data/local/tmp/sums.md5;`
1. Removes the sums file on the phone `adb shell "rm /data/local/tmp/sums.md5";`
1. Compares the local `.img.gz` backups to the ones creates on the phone `md5sum -c sums.md5`

As a note, all this complexity is required because:

1. Using the output of `adb shell` is not binary friendly, so
1. We require to pass the data with netcat, but
1. We cannot assume the phone and computer to be on the same fast LAN, hence
1. We neet to use the ADB forward feature.
1. Also, a backup without checksum cannot be trusted, however
1. While using the system, the content of some partitions changes, therefore
1. We need to read each partition only once, and `tee` the result to both netcat and md5sum. Surprisingly,
1. The shell in Android is the very old Bourne Shell and doesn't support process substitution, consequently,
1. We can use FIFOs to have the same behavior.
1. Finally, netcat wants a non-desirable user input to close the communication, so
1. We can work around it using a predefined string containing a Here Document, run by being piped to a shell interpreter

In other words, if ADB were binary friendly, netcat not requiring user input at any time, and Android shell less antique, it would be much shorter and easier to understand.

**However**, we need a final step: we have saved neatly and nicely all the partitions but one thing is missing. The final part is 
the partition map which is at the beginning of the disk. As far as I know, the following is enough (using fdisk as shown in the next section to adapt it: `bs` equal to the `Logical sector size` value, and `count` equal to first `Start (sector)` minus 1):

```console
adb shell "su -c dd if=/dev/block/mmcblk0 of=/data/local/tmp/mbr bs=512 count=33"; adb pull /data/local/tmp/mbr; adb shell "rm /data/local/tmp/mbr"
```

## More information about partitions

To know more about the partitions, check [*El Grande Partition Table Reference* in the **xda**developer forums](https://forum.xda-developers.com/showthread.php?t=1959445). I save the output here as I don't know if it will be useful in the future. Below is the `fdisk` output, ran on both the device and partitions (to get more info therwise fails). It has been restyled by hand to be a little more legible.

```console
root@android:/ # fdisk -l /dev/block/mmcblk*                     
Found valid GPT with protective MBR; using GPT

Disk /dev/block/mmcblk0: 15269888 sectors, 3360M
Logical sector size: 512
Disk identifier (GUID): 98101b32-bbe2-4bf2-a06e-2bb33d000c20
Partition table holds up to 28 entries
First usable sector is 34, last usable sector is 15269854

Number  Start (sector)    End (sector)  Size Name
     1              34             545  256K sbl1
     2             546            1057  256K sbl2
     3            1058            2081  512K sbl3
     4            2082            2593  256K rpm
     5            2594            4641 1024K tz
     6            4642            8737 2048K aboot
     7            8738           12833 2048K aboot2
     8           12834           12849  8192 ssd
     9           16384           24575 4096K fsg
    10           24576           32767 4096K modemst1
    11           32768           40959 4096K modemst2
    12           40960          106495 32.0M oeminfo
    13          106496          237567 64.0M modem
    14          237568          270335 16.0M recovery
    15          270336          303103 16.0M recovery2
    16          303104          311295 4096K misc
    17          311296          319487 4096K splash
    18          319488          335871 8192K persist
    19          335872          352255 8192K tombstones
    20          352256          353535  640K pad
    21          360448          385023 12.0M boot
    22          385024          909311  256M cust
    23          909312         1302527  192M cache
    24         1302528         3399679 1024M system
    25         3399680         5709823 1128M userdata
    26         5709824        15269854 4667M grow
Disk /dev/block/mmcblk0p1: 0 MB, 262144 bytes, 512 sectors
8 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p1 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p10: 4 MB, 4194304 bytes, 8192 sectors
128 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p10 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p11: 4 MB, 4194304 bytes, 8192 sectors
128 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p11 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p12: 32 MB, 33554432 bytes, 65536 sectors
1024 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p12 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p13: 64 MB, 67108864 bytes, 131072 sectors
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
Disk /dev/block/mmcblk0p14: 16 MB, 16777216 bytes, 32768 sectors
512 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p14 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p15: 16 MB, 16777216 bytes, 32768 sectors
512 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p15 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p16: 4 MB, 4194304 bytes, 8192 sectors
128 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p16 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p17: 4 MB, 4194304 bytes, 8192 sectors
128 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p17 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p18: 8 MB, 8388608 bytes, 16384 sectors
256 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p18 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p19: 8 MB, 8388608 bytes, 16384 sectors
256 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p19 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p2: 0 MB, 262144 bytes, 512 sectors
8 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p2 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p20: 0 MB, 655360 bytes, 1280 sectors
20 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p20 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p21: 12 MB, 12582912 bytes, 24576 sectors
384 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p21 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p22: 256 MB, 268435456 bytes, 524288 sectors
8192 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p22 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p23: 192 MB, 201326592 bytes, 393216 sectors
6144 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p23 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p24: 1024 MB, 1073741824 bytes, 2097152 sectors
32768 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p24 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p25: 1128 MB, 1182793728 bytes, 2310144 sectors
36096 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p25 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p26: 4667 MB, 4894735872 bytes, 9560031 sectors
149375 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Device                Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
Disk /dev/block/mmcblk0p3: 0 MB, 524288 bytes, 1024 sectors
16 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p3 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p4: 0 MB, 262144 bytes, 512 sectors
8 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p4 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p5: 1 MB, 1048576 bytes, 2048 sectors
32 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p5 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p6: 2 MB, 2097152 bytes, 4096 sectors
64 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p6 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p7: 2 MB, 2097152 bytes, 4096 sectors
64 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p7 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p8: 0 MB, 8192 bytes, 16 sectors
0 cylinders, 4 heads, 16 sectors/track
Units: cylinders of 64 * 512 = 32768 bytes

Disk /dev/block/mmcblk0p8 doesn't contain a valid partition table
Disk /dev/block/mmcblk0p9: 4 MB, 4194304 bytes, 8192 sectors
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
```

```console
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

```console
root@android:/ # cat /proc/diskstats                                         
   1       0 ram0 0 0 0 0 0 0 0 0 0 0 0
   1       1 ram1 0 0 0 0 0 0 0 0 0 0 0
   1       2 ram2 0 0 0 0 0 0 0 0 0 0 0
   1       3 ram3 0 0 0 0 0 0 0 0 0 0 0
   1       4 ram4 0 0 0 0 0 0 0 0 0 0 0
   1       5 ram5 0 0 0 0 0 0 0 0 0 0 0
   1       6 ram6 0 0 0 0 0 0 0 0 0 0 0
   1       7 ram7 0 0 0 0 0 0 0 0 0 0 0
   1       8 ram8 0 0 0 0 0 0 0 0 0 0 0
   1       9 ram9 0 0 0 0 0 0 0 0 0 0 0
   1      10 ram10 0 0 0 0 0 0 0 0 0 0 0
   1      11 ram11 0 0 0 0 0 0 0 0 0 0 0
   1      12 ram12 0 0 0 0 0 0 0 0 0 0 0
   1      13 ram13 0 0 0 0 0 0 0 0 0 0 0
   1      14 ram14 0 0 0 0 0 0 0 0 0 0 0
   1      15 ram15 0 0 0 0 0 0 0 0 0 0 0
   7       0 loop0 0 0 0 0 0 0 0 0 0 0 0
   7       1 loop1 0 0 0 0 0 0 0 0 0 0 0
   7       2 loop2 0 0 0 0 0 0 0 0 0 0 0
   7       3 loop3 0 0 0 0 0 0 0 0 0 0 0
   7       4 loop4 0 0 0 0 0 0 0 0 0 0 0
   7       5 loop5 0 0 0 0 0 0 0 0 0 0 0
   7       6 loop6 0 0 0 0 0 0 0 0 0 0 0
   7       7 loop7 0 0 0 0 0 0 0 0 0 0 0
 179       0 mmcblk0 56051 10089900 13722294 410080 1177 1875 24721 23720 0 218660 433560
 179       1 mmcblk0p1 1 63 512 10 0 0 0 0 0 10 10
 179       2 mmcblk0p2 1 63 512 0 0 0 0 0 0 0 0
 179       3 mmcblk0p3 2 126 1024 20 0 0 0 0 0 20 20
 179       4 mmcblk0p4 1 63 512 10 0 0 0 0 0 10 10
 179       5 mmcblk0p5 4 252 2048 40 0 0 0 0 0 40 40
 179       6 mmcblk0p6 45 2003 16384 740 0 0 0 0 0 600 740
 179       7 mmcblk0p7 59 1989 16384 680 0 0 0 0 0 390 680
 179       8 mmcblk0p8 1 1 16 10 0 0 0 0 0 10 10
 179       9 mmcblk0p9 31 993 8192 210 0 0 0 0 0 110 210
 179      10 mmcblk0p10 48 1802 14800 430 0 0 0 0 0 350 430
 179      11 mmcblk0p11 34 995 8232 200 49 720 6152 380 0 500 580
 179      12 mmcblk0p12 272 7941 65704 1660 0 0 0 0 0 920 1660
 179      13 mmcblk0p13 968 130552 220681 6490 0 0 0 0 0 3710 6490
 179      14 mmcblk0p14 108 3988 32768 690 0 0 0 0 0 470 690
 179      15 mmcblk0p15 127 3969 32768 850 0 0 0 0 0 450 850
 179      16 mmcblk0p16 23 1001 8192 170 0 0 0 0 0 140 170
 179      17 mmcblk0p17 29 995 8192 130 0 0 0 0 0 120 130
 179      18 mmcblk0p18 79 2025 16826 570 6 1 56 10 0 320 580
 179      19 mmcblk0p19 63 1985 16384 400 0 0 0 0 0 230 400
 179      20 mmcblk0p20 4 156 1280 30 0 0 0 0 0 20 30
 179      21 mmcblk0p21 220 12068 98304 1810 0 0 0 0 0 1650 1810
 179      22 mmcblk0p22 2131 63543 529610 21360 2 0 16 0 0 11310 21350
 179      23 mmcblk0p23 1548 47630 393418 10160 8 2 80 20 0 5370 10170
 179      24 mmcblk0p24 11666 254413 2453066 76140 19 10 232 60 0 37160 76150
 179      25 mmcblk0p25 1403 23529 211484 6800 989 1142 18184 23210 0 19820 30010
 179      26 mmcblk0p26 37177 9527752 9564929 280460 1 0 1 30 0 148470 280320
```

{% include_relative toc.md %}
