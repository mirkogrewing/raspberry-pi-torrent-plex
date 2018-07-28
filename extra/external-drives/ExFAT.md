# Mounting an ExFAT Formatted Drive

## Requirements

1. An ExFAT formatted USB Drive

## Preparation

Plug the USB drive to your Raspberry Pi and restart.

From the terminal execute:

```
$ sudo fdisk -l
```

Which will list all the partitions recognised by the system. Identify the one associated with the external drive. In my case it was:

```
Disk /dev/sda: 1.8 TiB, 2000398934016 bytes, 3907029168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

Device     Boot Start        End    Sectors  Size Id Type
/dev/sda1           2 3907029167 3907029166  1.8T  7 HPFS/NTFS/exFAT
```

Now install the exFAT drivers, otherwise the file-system will not be recognised:

```
$ sudo apt-get install exfat-fuse
```

and create the directory where the disk will be mounted:

```
$ sudo mkdir /media/storage
```

## Mount

Now you can mount the disk with:

```
$ sudo mount /dev/sda1/ /media/storage/
```

Please remember to replace `sda1` with the actual device name that you found with `sudo fdisk -l`.

To unmount:

```
$ sudo umount /media/storage
```

### Mount at Boot

Ideally, we want the drive to be always connected, so we configure the mount at boot.

First we find the identifier:

```
$ sudo blkid
```

In my case the result was:

```
/dev/mmcblk0p1: LABEL="boot" UUID="3725-1C05" TYPE="vfat" PARTUUID="e6a86318-01"
/dev/mmcblk0p2: LABEL="rootfs" UUID="fd695ef5-f047-44bd-b159-2a78c53af20a" TYPE="ext4" PARTUUID="e6a86318-02"
/dev/mmcblk0: PTUUID="e6a86318" PTTYPE="dos"
/dev/sda1: LABEL="Storage" UUID="5B57-8994" TYPE="exfat"
```

What we are interested in is the UUID of our external drive. In my case that's "5B57-8994".

Now we can edit `/etc/fstab` and add this line at the bottom:

```
UUID=5B57-8994 /media/storage exfat defaults,auto,umask=000,users,rw 0 0
```

Don't forget to replace the UUID with yours.