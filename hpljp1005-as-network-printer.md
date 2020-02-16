---
layout: default
title: HP LaserJet P1005 as Network Printer
---
## HP LaserJet P1005 as Network Printer

### Generic Print Server Setup

* Setup Raspbian on SD card as described [here](setup-raspbian-on-sd-card), but change the device name to **printerpi**.

* SSH to device
```
$ ssh pi@printerpi
```

* Create the **lpadmin** group, if it doesn't exist
```
$ sudo groupadd lpadmin
```

* Add the **pi** user into the **lpadmin** group
```
$ sudo usermod -a -G lpadmin pi
```

* Update and upgrade packages
```
$ sudo apt update
$ sudo apt upgrade
```

* Install [CUPS](https://en.wikipedia.org/wiki/CUPS). It allows a computer to act as a print server
```
$ sudo apt install cups
```

* Replace **/etc/cusp/cupsd.conf** with the code below:
```
  $ sudo tee /etc/cups/cupsd.conf <<EOF
  LogLevel warn
  PageLogFormat
  MaxLogSize 0
  Listen 0.0.0.0:631
  Listen /var/run/cups/cups.sock
  Browsing On
  BrowseLocalProtocols dnssd
  DefaultAuthType Basic
  WebInterface Yes
  <Location />
    Order allow,deny
    Allow @Local
  </Location>
  <Location /admin>
    Order allow,deny
    Allow @Local
  </Location>
  <Location /admin/conf>
    AuthType Default
    Require user @SYSTEM
    Order allow,deny
    Allow @Local
  </Location>
  <Location /admin/log>
    AuthType Default
    Require user @SYSTEM
    Order allow,deny
    Allow @Local
  </Location>
  <Policy default>
    JobPrivateAccess default
    JobPrivateValues default
    SubscriptionPrivateAccess default
    SubscriptionPrivateValues default
    <Limit Create-Job Print-Job Print-URI Validate-Job>
      Order deny,allow
    </Limit>
    <Limit Send-Document Send-URI Hold-Job Release-Job Restart-Job Purge-Jobs Set-Job-Attributes Create-Job-Subscription Renew-Subscription Cancel-Subscription Get-Notifications Reprocess-Job Cancel-Current-Job Suspend-Current-Job Resume-Job Cancel-My-Jobs Close-Job CUPS-Move-Job CUPS-Get-Document>
      Require user @OWNER @SYSTEM
      Order deny,allow
    </Limit>
    <Limit CUPS-Add-Modify-Printer CUPS-Delete-Printer CUPS-Add-Modify-Class CUPS-Delete-Class CUPS-Set-Default CUPS-Get-Devices>
  #    Keeps asking for user/password when adding printer
  #    AuthType Default
  #    Require user @SYSTEM
      Order deny,allow
    </Limit>
    <Limit Pause-Printer Resume-Printer Enable-Printer Disable-Printer Pause-Printer-After-Current-Job Hold-New-Jobs Release-Held-New-Jobs Deactivate-Printer Activate-Printer Restart-Printer Shutdown-Printer Startup-Printer Promote-Job Schedule-Job-After Cancel-Jobs CUPS-Accept-Jobs CUPS-Reject-Jobs>
      AuthType Default
      Require user @SYSTEM
      Order deny,allow
    </Limit>
    <Limit CUPS-Authenticate-Job>
      Require user @OWNER @SYSTEM
      Order deny,allow
    </Limit>
    <Limit All>
      Order deny,allow
    </Limit>
  </Policy>
  <Policy authenticated>
    JobPrivateAccess default
    JobPrivateValues default
    SubscriptionPrivateAccess default
    SubscriptionPrivateValues default
    <Limit Create-Job Print-Job Print-URI Validate-Job>
      AuthType Default
      Order deny,allow
    </Limit>
    <Limit Send-Document Send-URI Hold-Job Release-Job Restart-Job Purge-Jobs Set-Job-Attributes Create-Job-Subscription Renew-Subscription Cancel-Subscription Get-Notifications Reprocess-Job Cancel-Current-Job Suspend-Current-Job Resume-Job Cancel-My-Jobs Close-Job CUPS-Move-Job CUPS-Get-Document>
      AuthType Default
      Require user @OWNER @SYSTEM
      Order deny,allow
    </Limit>
    <Limit CUPS-Add-Modify-Printer CUPS-Delete-Printer CUPS-Add-Modify-Class CUPS-Delete-Class CUPS-Set-Default>
      AuthType Default
      Require user @SYSTEM
      Order deny,allow
    </Limit>
    <Limit Pause-Printer Resume-Printer Enable-Printer Disable-Printer Pause-Printer-After-Current-Job Hold-New-Jobs Release-Held-New-Jobs Deactivate-Printer Activate-Printer Restart-Printer Shutdown-Printer Startup-Printer Promote-Job Schedule-Job-After Cancel-Jobs CUPS-Accept-Jobs CUPS-Reject-Jobs>
      AuthType Default
      Require user @SYSTEM
      Order deny,allow
    </Limit>
    <Limit Cancel-Job CUPS-Authenticate-Job>
      AuthType Default
      Require user @OWNER @SYSTEM
      Order deny,allow
    </Limit>
    <Limit All>
      Order deny,allow
    </Limit>
  </Policy>
  EOF
```

* Enabled and restart the **CUPS** service
```
$ sudo systemctl enable cups
$ sudo systemctl restart cups
```

* Install [Foomatic](https://wiki.linuxfoundation.org/openprinting/database/foomatic),
which is a free database for printer drivers
```
$ sudo apt install foomatic-db foomatic-db-engine
```

* Install [Samba](https://www.samba.org). It allows Unix and Linux computers to share directories and printers  with Windows computers over the network. Installing Samba is not strictly necessary, but it's a valuable option.
```
$ sudo apt install samba
```

* Ensure the file **/etc/samba/smb.conf** has the code below:
```
[printers]
    comment = All Printers
    browseable = no
    path = /var/spool/samba
    printable = yes
    guest ok = yes
    read only = yes
    create mask = 0700
```

* Enabled and restart the **Samba** service
```
$ sudo systemctl enable smbd
$ sudo systemctl enable nmbd
$ sudo systemctl restart smbd
$ sudo systemctl restart nmbd
```

**NB**: You can find the above setup automated using [Ansible](https://www.ansible.com/) [here](https://github.com/ten0s/rpi).

### Setup HP LaserJet P1005

* Plug-in and switch on the printer
* List available firmware files
```
$ sudo getweb
...
    $ ./getweb p1005	# Get HP LJ P1005 firmware file
...
```
* Get HP LaserJet P1005 firmware file
```
$ sudo getweb p1005
```
* Restart the printer
* In the browser go to [**CUPS** admin page](http://printerpi:631/admin)
* Ensure **Allow users to cancel any job (not just their own)** is checked.
* Click **Add Printer**
* Choose **Local Printers**: HP LaserJet P1005 (HP LaserJet P1005)
* Click **Continue**
* Enable **Share This Printer**
* Click **Continue**
* Choose **Model**: HP LaserJet P1005 Foomatic/foo2xqx (recommended) (en)
* Click **Add Printer**
* Click **Set Default Options**
* Click **Maintenance** and then **Print Test Page**

That's it! Now you have a fully functional print server accessible via [Internet Printing Protocol (IPP)](https://en.wikipedia.org/wiki/Internet_Printing_Protocol) and [Samba](https://www.samba.org).
