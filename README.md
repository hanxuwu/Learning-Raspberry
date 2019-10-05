# Learning-Raspberry

## Tutorial


1. Using the tools/guiformat.exe format the sdcard

Note if you using the micro SD(SDXC) size > 64G,you have to using this tool,because exfat is not support by Raspberry Pi 4,should using FAT32


2. Download the NOOBS_v3_2_0 copy to micro SD(SDXC)


## Command

1. Check the temperature
```s
pi@RaspberryPi:~ $ /opt/vc/bin/vcgencmd measure_temp
```

2. Speed performance test
```s
curl https://raw.githubusercontent.com/TheRemote/PiBenchmarks/master/Storage.sh | sudo bash
```

3. sysbench

a. calculate primes
```s
sysbench --test=cpu --cpu-max-prime=20000 run
```
b. test the I/O Output of your Raspberry Pi
```s
sysbench --test=fileio --file-total-size=2G prepare
sysbench --test=fileio --file-total-size=2G --file-test-mode=rndrw --init-rng=on --max-time=300 --max-requests=0 run
sysbench --test=fileio --file-total-size=2G cleanup
```

c. memory read and write
```s
sysbench --test=memory run --memory-total-size=2G
sysbench --test=memory run --memory-total-size=2G --memory-oper=read
```


## Raspberry Pi 4B  U-disk/SSD BOOT

1. Prepare Bootloader SD Card – Image your SD card with the latest Raspbian 10 “Buster” release (I prefer Raspbian Lite) however you would normally do it.

2. Prepare SSD / Flash Drive – Image your SSD or Flash Drive. Make sure you create the empty file named “ssh” on the boot partition of both drives.

Using https://www.balena.io/etcher/ image to ssd

Note:Noob image doesn't work

3. update the system
```s
sudo apt-get update && sudo apt-get dist-upgrade
```

4.run blkid
```s
sudo blkid
```

```s
/dev/mmcblk0p1: LABEL_FATBOOT="boot" LABEL="boot" UUID="3FFE-CDCA" TYPE="vfat" PARTUUID="eed33d96-01"
/dev/mmcblk0p2: LABEL="rootfs" UUID="3122c401-b3c6-4d27-8e0d-6708a7613aed" TYPE="ext4" PARTUUID="eed33d96-02"
/dev/sda1: LABEL_FATBOOT="boot" LABEL="boot" UUID="3FFE-CDCA" TYPE="vfat" PARTUUID="698e33bd-01"
/dev/sda2: LABEL="rootfs" UUID="3122c401-b3c6-4d27-8e0d-6708a7613aed" TYPE="ext4" PARTUUID="698e33bd-02"
/dev/mmcblk0: PTUUID="eed33d96" PTTYPE="dos"
```
what we need is the PTUUID of `sda2`


```s
/dev/sda2: LABEL="rootfs" UUID="3122c401-b3c6-4d27-8e0d-6708a7613aed" TYPE="ext4" PARTUUID="698e33bd-02"
```

5. edit /boot/cmdline.txt

a. backup first

```s
sudo cp /boot/cmdline.txt /boot/cmdline.txt.bak
```
run
```s
sudo nano /boot/cmdline.txt
```
before
```s
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=eed33d96-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles

```
after:
```s
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=698e33bd-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles

```

6. reboot 
```s
findmnt -n -o SOURCE /
```
if display */dev/sda2* means success





7. subsitude ssd boot to sdcard boot *PARTUUID*

```s
sudo nano /etc/fstab
```

before
```s
proc                  /proc           proc    defaults          0       0
PARTUUID=698e33bd-01  /boot           vfat    defaults          0       2
PARTUUID=698e33bd-02  /               ext4    defaults,noatime  0       1
```
after

```s
proc                  /proc           proc    defaults          0       0
PARTUUID=eed33d96-01  /boot           vfat    defaults          0       2
PARTUUID=698e33bd-02  /               ext4    defaults,noatime  0       1
```



8.  Adjust file system size

```S
pi@raspberrypi:~ $ sudo fdisk /dev/sda
```

```s
Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk /dev/sda: 238.5 GiB, 256060514304 bytes, 500118192 sectors
Disk model: Generic         
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x698e33bd

Device     Boot  Start      End  Sectors  Size Id Type
/dev/sda1         8192   532480   524289  256M  c W95 FAT32 (LBA)
/dev/sda2       540672 12566527 12025856  5.8G 83 Linux

```

/dev/sda2（rootfs）start from 540672, we need to delete the old and create a new large partition,don't make any mistake in following steps.

```s
Command (m for help): d
Partition number (1,2, default 2): 2
Partition 2 has been deleted.
Command (m for help): n
Partition type
    p   primary (1 primary, 0 extended, 3 free)
    e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (532481-500118191, default 589815): 540672 (enter the start value exactly as it was, the default will be wrong)
Last sector, +/-sectors or +/-size{K,M,G,T,P} (540672-500118191, default 500118191): (press enter to accept default which is the full disk)
Created a new partition 2 of type 'Linux' and of size 238.2 GiB.
Partition #2 contains a ext4 signature.
Do you want to remove the signature? [Y]es/[N]o: n 
Command (m for help): w
```

9. Restart the system 
```s
sudo resize2fs /dev/sda2
```


> resize2fs 1.44.5 (15-Dec-2018)
> Filesystem at /dev/sda2 is mounted on /; on-line resizing required
> old_desc_blocks = 1, new_desc_blocks = 15
> The filesystem on /dev/sda2 is now 62447190 (4k) blocks long.


finish extend the partition,check it using
```s
df -h
```

10. Test the performance  

```s
curl https://raw.githubusercontent.com/TheRemote/PiBenchmarks/master/Storage.sh | sudo bash
```


Using macos disk utility format the ssd disk 
```s
ls -l /dev/disk/by-uuid
```

Install
```s
wget https://packagecloud.io/headmelted/codebuilds/gpgkey -O -| sudo apt-key
```





References：
https://www.quarkbook.com/?p=638  
https://raspberrypi.stackexchange.com/questions/3371/how-can-i-stress-test-my-raspberry-pi  
https://sites.google.com/site/zsgititit/home/raspberry-shu-mei-pai/raspberry-shi-yong64g-sdxc-micro-sd-ka  
https://www.raspberrypi.org/downloads/noobs/
https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/