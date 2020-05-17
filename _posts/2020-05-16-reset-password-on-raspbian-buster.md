---
layout: post
title: Reset Password on Raspbian Buster
date: 2020-05-16
---
I recently replaced my laptop, changed ssh key and, as it happens, forgot the password, so I can't ssh to Raspberry Pi any more.

## TL;DR

Here are quick steps how to reset the password. Later I will describe what they mean and add some more advanced commands.

* Turn off Raspberry Pi
* Put the SD card into the card reader
* Add to the end of **cmdline.txt**:

```
 rw init=/bin/sh
```

* Put the SD card into Raspberry Pi 
* Reboot
* Change the user **pi** password:

```
# passwd pi
New password:
Retype new password:
passwd: password updated successfully
```

* Flush the file system cache 

```
# sync
```

* Turn off Raspberry Pi
* Put the SD card into the card reader and return **cmdline.txt** into the initial state by removing:

```
 rw init=/bin/sh
```

* Put the SD card into Raspberry Pi
* Reboot

If it's everything you needed you can stop reading.

## rw

```
 rw init=/bin/sh
```

The **rw** flag mounts the root partition **/** as **r**ead-**w**rite. If you omit it,
the root partition **/** will be read-only and changing password will fail:

```
# passwd pi
Enter new password:
Retype new password:
passwd: Authentication token manipulation error
passwd: password unchanged
```

To remount it as **r**ead-**w**rite run:

```
# mount -o remount, rw /
EXT4-fs (mmcblk0p2): re-mounted.
```

## init=/bin/sh

The **init** flag specifies the program to run as the [Init](https://en.wikipedia.org/wiki/Init) process, i.e. the first process to be run. Here we replace the default **/sbin/init**, which is responsible for running all the init scripts with **/bin/sh**, which runs a simple shell.

## /boot/cmdline.txt

The file **cmdline.txt** needs to be edited back, but it's on the **/boot** partition, which is not mounted by default. Still, it's possible to do it without moving the SD card around. 

* Determine the **/boot** partition device:

```
# blkid
/dev/mmcblk0p1 ... LABEL="boot" TYPE="vfat"
/dev/mmcblk0p2 ... LABEL="rootfs" TYPE="ext4"
```

* Mount the **/boot** partition:

```
# mount /dev/mmcblk0p1 /boot
```

* Open the file for editing:

```
# nano /boot/cmdline.txt
```

* Remove the below fragment, save and exit the editor:

```
 rw init=/bin/sh
```

* Flush the file system cache:

```
# sync /boot
```

* Unmount the **/boot** partition:

```
# umount /boot
```

* Reboot

## References:

* http://mapledyne.com/ideas/2015/8/4/reset-lost-admin-password-for-raspberry-pi
* https://www.raspberrypi.org/forums/viewtopic.php?t=20397
* https://www.redhat.com/archives/rhl-list/2005-March/msg04089.html
