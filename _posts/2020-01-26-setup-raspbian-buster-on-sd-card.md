---
layout: post
title: Setup Raspbian Buster on SD card
date: 2020-01-26
---
* Download the latest image from [https://www.raspberrypi.org/downloads/raspbian/](https://www.raspberrypi.org/downloads/raspbian/)
```
$ wget --trust-server-names https://downloads.raspberrypi.org/raspbian_lite_latest
```

* Ensure checksum is correct
```
$ sha256sum 2020-02-05-raspbian-buster-lite.zip
7ed5a6c1b00a2a2ab5716ffa51354547bb1b5a6d5bcb8c996b239f9ecd25292b  2020-02-05-raspbian-buster-lite.zip
```

* Unzip image
```
$ unzip 2020-02-05-raspbian-buster-lite.zip
```

* Insert SD card into card reader

* Determine SD card device name
```
$ sudo fdisk -l
...
Disk /dev/mmcblk0: 31.3 GB
...
```

* Un-mount SD card (if gets mounted automatically)
```
$ sudo umount /dev/mmcblk0p*
```

* Copy image to SD card
```
$ sudo dd if=2020-02-05-raspbian-buster-lite.img of=/dev/mmcblk0 bs=4M status=progress conv=fsync
```

* Determine SD card partition names
```
$ sudo fdisk -l
...
Disk /dev/mmcblk0: 31.3 GB
...
Device         Boot  Start     End Sectors  Size Id Type
/dev/mmcblk0p1        8192  532479  524288  256M  c W95 FAT32 (LBA)
/dev/mmcblk0p2      532480 4390911 3858432  1.9G 83 Linux
...
```

* Un-mount SD card (if gets mounted automatically)
```
$ sudo umount /dev/mmcblk0p*
```

* Mount SD card boot partition
```
$ sudo mkdir -p /mnt/boot
$ sudo mount -t vfat /dev/mmcblk0p1 /mnt/boot
```

* Enable SSH
```
$ sudo touch /mnt/boot/ssh
```

* Enable WIFI (optional). Replace \<SSID\> and \<SECRET\> with correct values
```
$ sudo tee /mnt/boot/wpa_supplicant.conf << EOF
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    ssid="<SSID>"
    psk="<SECRET>"
    scan_ssid=1
    key_mgmt=WPA-PSK
}
EOF
```

* Un-mount SD card boot partition
```
$ sudo umount /mnt/boot
```

* Remove SD card from card reader
* Insert SD card into Raspberry Pi
* Plug-in LAN cable (or use WIFI) and power supply.
* After a couple of minutes you should be able to ping
```
$ ping -c 1 raspberrypi
PING raspberrypi (192.168.100.33) 56(84) bytes of data.
64 bytes from raspberrypi (192.168.100.33): icmp_seq=1 ttl=64 time=5.69 ms
...
```

* Generate your SSH key pair (if you don't have one)
```
$ ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
```

* Copy your SSH public key. Default password for the **pi** user is **raspberry**
```
$ ssh -F /dev/null pi@raspberrypi -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null 'mkdir -p ~/.ssh/; cat >> ~/.ssh/authorized_keys' < ~/.ssh/id_rsa.pub
```

* Now you should be able to SSH without password
```
$ ssh pi@raspberrypi
```

* Configure
```
$ sudo raspi-config
```
  * Change User Password (passwd)
  * Network Options
    * N1 Hostname (/etc/hostname, /etc/hosts)
  * Localization Options
    * I1 Change Locale: en_US.UTF-8 UTF-8
    * I2 Change Timezone (/etc/localtime)

* Reboot
```
$ sudo reboot
```
