# File Sharing #

When you plug an external hard disk, it'll be mounted under **/mnt/** using its **label**, for partition without label will mounted as **hdusb-sdX**

**/mnt** and all of its content is ready to be accessed by services below:
  * Web browser (read only) http://alix.site/ or https://alix.site/mnt/
  * Win share (read write) \\alix\mnt
  * FTP client (read write) [ftp://alix.site:2121](ftp://alix.site:2121) (see comments below)
  * Secure FTP (ready write) sftp://alix.site . Its run over SSH, you can use root login to access entire system. FileZilla is very good sftp client._* Media Server (read only) use any UPnP/DLNA compatible client._

FTP server it's configured to accept authenticated user only, and they are restricted to their home directory, then you can create many users you need, each one seeing what you want.

_Security tip: creating users with **-s /bin/false**, they won´t be able to access the system through SSH terminal or sftp_


  * Volatile users. It´ll be lost on next boot, also recommended when you plan add an user for a couple transfers and just after it´ll be removed.
```
useradd -d /mnt -s /bin/false ftpmaster
useradd -d /mnt/somedrive/somefolder -s /bin/false ftprestrict
passwd ftpmaster
passwd ftprestrict
```

  * Persistent users (really saved on CF disk)
```
remountrw
rebind on
chroot /ro

useradd -d /mnt -s /bin/false ftpmaster
useradd -d /mnt/somedrive/somefolder -s /bin/false ftprestrict
passwd ftpmaster
passwd ftprestrict

history -c ; exit
rebind off
remountro
```