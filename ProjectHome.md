It's a Debian image ready to use on ALIX board, just download and write on 2G (or large) compact flash.

Below, some features/services:

-- **read only file system, it protects against power failures and substantially increases compact flash life time**

-- **common services available on first boot**
  * serial terminal ( 38400n8 )
  * dnsmasq ( dns and dhcp servers )
  * iptables ( basic firewall rules and internet sharing )
  * samba ( MS Win file share )
  * cups ( print server )
  * vsftpd ( ftp server )
  * nginx with fastcgi ( http server )
  * minidlna ( media server )
  * openssh ( terminal and sftp )
  * stunnel ( ssl engine, https support pre-configured for nginx and transmission )
  * tinyproxy (http/https proxy server)
  * pptpd ( vpn server, MS Win has builtin client support )
  * snmpd ( snmp server )
  * openvpn (vpn server and client)
  * openconnect ( vpn client, cisco compatible )
  * external storage automount and sharing
  * basic web admin panel
  * performance monitor (on web panel)

-- **additional services available, just turned on**
  * transmission ( torrent p2p )
  * hostapd ( wireless access point / it needs a miniPCI card or usb dongle )

-- **Regular system maintenance through apt-get, use it to update, install and remove applications and patches**

-- **Keep in mind that it's a Debian i386, so any available software for that platform can be installed, regards hardware power limitations**

## **Image files are available for download at** [GoogleDrive](https://drive.google.com/folderview?id=0B9AGLVDYncLIYlU3SjBVSENXRTg&usp=sharing) ##

```
sha1sum:
b1de41920a4b16efb499b668d739c6d21a359aed *debian-for-alix-v1.img.bz2
ec291aa59295ce4737277581b86d98dfa31aaf71 *debian-for-alix-v2.img.bz2
f255435e7d5694b95ad4951f496f5e6f57121bff *debian-for-alix-v3.img.bz2
```


-- **debian-for-alix-v1.img.bz2 (debian 6 based)**

[version 1 wiki page](http://code.google.com/p/debian-for-alix/wiki/InstalationAndFirstSteps)

-- **debian-for-alix-v2.img.bz2 (debian 6 based)**

[version 2 wiki page](http://code.google.com/p/debian-for-alix/wiki/AlternateVersion)

-- **debian-for-alix-v3.img.bz2 (debian 7 based)**

Debian basic setup, only ssh server and basic utilities, also serial terminal was prepared, but no readonly file system and other features.

  * eth0: DHCP
  * eth1: static 172.16.210.254
  * eth2: no configuration
  * user: alix (root has locked password)
  * pass: 123456

By serial terminal or over network (ssh service) to log on system use 'alix' account then run 'sudo su -' to get a root privileged session.




Enjoy!

[![](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=meguerche%40gmail%2ecom&lc=US&item_name=Debian%20For%20ALIX%20%2d%20Donate&no_note=0&currency_code=USD&bn=PP%2dDonationsBF%3abtn_donateCC_LG%2egif%3aNonHostedGuest)


![http://debian-for-alix.googlecode.com/files/ALIX_BACK_OPENED_CASE.jpg](http://debian-for-alix.googlecode.com/files/ALIX_BACK_OPENED_CASE.jpg)

![http://debian-for-alix.googlecode.com/files/ScreenShot-WebAdm-System-info.png](http://debian-for-alix.googlecode.com/files/ScreenShot-WebAdm-System-info.png)

![http://debian-for-alix.googlecode.com/files/ScreenShot-WebAdm-System-monitor.png](http://debian-for-alix.googlecode.com/files/ScreenShot-WebAdm-System-monitor.png)

![http://debian-for-alix.googlecode.com/files/ScreenShot-WebAdm-Maintenance.png](http://debian-for-alix.googlecode.com/files/ScreenShot-WebAdm-Maintenance.png)

![http://debian-for-alix.googlecode.com/files/ScreenShot-CUPS.png](http://debian-for-alix.googlecode.com/files/ScreenShot-CUPS.png)

![http://debian-for-alix.googlecode.com/files/ScreenShot-CUPS-W7-PRINTER-CLIENT.png](http://debian-for-alix.googlecode.com/files/ScreenShot-CUPS-W7-PRINTER-CLIENT.png)

![http://debian-for-alix.googlecode.com/files/ScreenShot-WebAdm-TransmissionWeb.png](http://debian-for-alix.googlecode.com/files/ScreenShot-WebAdm-TransmissionWeb.png)

![http://debian-for-alix.googlecode.com/files/ScreenShot-Console-Serial-PuTTY.png](http://debian-for-alix.googlecode.com/files/ScreenShot-Console-Serial-PuTTY.png)

![http://debian-for-alix.googlecode.com/files/ScreenShot-WebAdm-Storage-mount.png](http://debian-for-alix.googlecode.com/files/ScreenShot-WebAdm-Storage-mount.png)

![http://debian-for-alix.googlecode.com/files/ScreenShot-CIFSandDLNA.png](http://debian-for-alix.googlecode.com/files/ScreenShot-CIFSandDLNA.png)