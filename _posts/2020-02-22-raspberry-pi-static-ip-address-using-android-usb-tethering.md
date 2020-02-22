---
layout: post
title: Raspberry Pi Static IP Address using Android USB Tethering
date: 2020-02-22
---
Recently I deployed my first Raspberry Pi [project]({{ site.baseurl }}{% post_url 2020-02-16-hplj1000-and-windows10 %}) into production :).
Shortly after doing that I thought - what am I going to do if something goes wrong: Wi-Fi router replaced or its password changed and
there's no a RJ45 patch cable to connect Raspberry Pi to the router, I bought a new phone and Bluetooth pairing is not valid any more, I don't
have a laptop or a card reader, etc.

How to connect to Raspberry Pi if all I have is my Android phone and a ubiquitous USB cable?

[Android Raspberry Pi display over USB](https://joshuawoehlke.com/android-raspberry-pi-display-over-usb/) got my attention.
It proposed to use [VNC Viewer](https://www.raspberrypi.org/documentation/remote-access/vnc/) on Android over USB tethering and
setup a static IP address on the usb0 network interface. It's usually better to avoid static IP addresses, but it's a great fit in this case.
The steps are hopelessly outdated there and don't work since Raspbian Jessie in 2016.
[How to give your Raspberry Pi a Static IP Address](https://thepihut.com/blogs/raspberry-pi-tutorials/how-to-give-your-raspberry-pi-a-static-ip-address-update)
helped me a lot. It took me a while until I made it work (I missed an equal sign in the config :). But playing with Android USB tethering
I noticed that even though Raspberry Pi's IP (usb0) changed constantly, the phone's IP (router) always stayed the same **192.168.42.129** even after reboots.
It turned out, this IP is indeed hardcoded as
[USB_NEAR_IFACE_ADDR](https://android.stackexchange.com/questions/46499/how-configure-the-dhcp-settings-of-wifi-tethering-hotspot-on-android).
This makes the below config pretty stable, at least until the phone maker doesn't change it.

So the config is:

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

Now plug-i USB cable into Raspberry Pi and Android phone, switch on
[USB tethering](https://www.instructables.com/id/Android-USB-Tethering/) and you know for sure
that Raspberry Pi is accessible from the phone by **192.168.42.42**.
Now use whatever protocol is appropriate for your use case **SSH**, **VNC**, **HTTP**, etc.

As usual ansible setup is [here](https://github.com/ten0s/rpi/tree/buster/roles/usb-static-ip).
