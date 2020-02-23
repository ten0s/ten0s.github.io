---
layout: post
title: Raspberry Pi Static IP Address for Android USB Tethering
date: 2020-02-22
---
Recently I deployed my first Raspberry Pi into
[production]({{ site.baseurl }}{% post_url 2020-02-16-hplj1000-and-windows10 %}) :).
Shortly after doing that I asked myself - what am I going to do if something goes wrong? What can go wrong?
Wi-Fi router replaced or its password changed and there's no a RJ45 patch cable to connect Raspberry Pi to the router.
There's no a laptop or a card reader. Etc.

How to connect to Raspberry Pi if all I have is my Android phone and a USB cable?

The article [Android Raspberry Pi display over USB](https://joshuawoehlke.com/android-raspberry-pi-display-over-usb/)
interested me. It proposed to use [VNC Viewer](https://www.raspberrypi.org/documentation/remote-access/vnc/)
on Android over USB tethering and setup a static IP address for the usb0 interface.
It's usually better to avoid static IP addresses, but it's a great fit in this case.
The steps are hopelessly outdated there and don't work since Raspbian Jessie in 2016.
It took me a while until I made it work. The article
[How to give your Raspberry Pi a Static IP Address](https://thepihut.com/blogs/raspberry-pi-tutorials/how-to-give-your-raspberry-pi-a-static-ip-address-update)
helped me a lot.  While playing with Android USB tethering I noticed that even though Raspberry Pi's IP
would changed constantly, the phone's IP **192.168.42.129** (router) always stayed the same even after phone reboots.
The IP from the very middle of the range **192.168.42.*** looked very suspicious. It turned out, the IP is indeed hardcoded as
[USB_NEAR_IFACE_ADDR](https://android.stackexchange.com/questions/46499/how-configure-the-dhcp-settings-of-wifi-tethering-hotspot-on-android).
This makes the below config pretty stable, at least until the phone maker doesn't change it.

The steps are:

* Leave **/etc/network/interfaces** as it is

* Add to **/etc/dhcpcd.conf**
```
interface usb0
static ip_address=192.168.42.42/24
static routers=192.168.42.129
static domain_name_servers=192.168.42.129
```

* Restart the **Dhcpcd** service
```
$ sudo systemctl restart dhcpcd
```

* Connect Raspberry Pi and Android phone using the USB cable

* Turn on [USB tethering](https://www.instructables.com/id/Android-USB-Tethering/)

That's it! Raspberry Pi is accessible from the phone by **192.168.42.42**.
Now use whatever protocol is appropriate: **SSH**, **VNC**, **HTTP**, etc.

You can find the [Ansible](https://www.ansible.com/) role [here](https://github.com/ten0s/rpi/tree/buster/roles/usb-static-ip).