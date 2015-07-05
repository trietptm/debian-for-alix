# Instalation and first steps

# Introduction #
## debian-for-alix-v1.img.bz2 ##

All you need to get your system runnning, in a couple commands.


# writing image on CF #

  * **download the image file**
  * **verify the checksum of debian-alix-v1.img.bz2** _this step is not required, but recommended to avoid broken files._
  * **write image on a 2GB (or large) compact flash**

example
```
sha1sum debian-alix-v1.img.bz2

bzcat debian-alix-v1.img.bz2 | dd of=/dev/sdb bs=16k
```
_The device path perhaps can be other like /dev/sdc, /dev/sdd ... according your environment_

# first boot, pre-configured settings #

**ALIX2D13** and **ALIX2D3** (3 RJ45 connectors)

**ALIX2D2** also works well without changes, obviously **eth2** won't load, but LAN configuration was made over **br0**

![http://debian-for-alix.googlecode.com/files/ALIX2D13_LAN_SETTINGS.jpg](http://debian-for-alix.googlecode.com/files/ALIX2D13_LAN_SETTINGS.jpg)

  * serial port: 38400n8
  * WAN IP: dhcp ( **eth0** )
  * LAN IP: **172.16.210.254**/255.255.255.0 ( **br0** => **eth1** and **eth2** )
  * hostname: alix.site
  * ssh server: 22 (default port)
  * **root** password: **123456**
  * char set: en\_US.UTF8
  * LAN dhcp range: 172.16.210.121 - 172.16.210.200
  * VPN pptp range: 172.16.210.201 - 172.16.210.220
  * WLAN: disable by default, because many people doesn't have one

# save your changes #

The **/** partition is an union of **/ro** (persistent / CF Card) and **/rw** (volatile / RAM), then by default all of your changes over **/** will stored on **/rw**, keeping **/ro** untouched, so changes will be lost each boot.

I'll show you two ways to persist you changes:

**1: move changed files from volatile to persistent partition**
  * do changes normally over **/** structure (edit files, run commands, restart services, do tests)
  * run **remountrw** to turn **/ro** writable
  * **move** changed files **from** /rw **to** /ro
  * run **remountro** to turn **/ro** read only again

example
```
nano /etc/default/transmission-daemon
nano /var/lib/transmission/settings.json.template
remountrw
mv /rw/etc/default/transmission-daemon /ro/etc/default/
mv /rw/var/lib/transmission/settings.json.template /ro/var/lib/transmission/
remountro
```

**2: use chroot environment to write direct over /ro partition**

_recommended to install/update/remove applications and complex changes involving many files_

  * run **remountrw** to turn **/ro** writable
  * run **rebind on** to bind system partitions under **/ro**
  * run **chroot /ro** to turn "/ro" your root environment
  * do your changes, they'll be saved direct on CF Card
  * run **history -c ; exit** to erase command history and get out chrooted environment, backing to the true **/** environment
  * run **rebind off** to disassemble system partitions under **/ro**
  * run **remountro** to turn **/ro** read only again

example
```
remountrw
rebind on
chroot /ro
apt-get update
apt-get -y dist-upgrade
apt-get clean
history -c ; exit
rebind off
remountro
```

_when your changes restart some service while you are inside chroot environment, you'll probably need run **lsof /ro** to find process that are locking resources and kill them before successfully use **rebind off** and **remountro**_

_If commands before don't help you, at last option run **mount -o remount,ro -f /ro** and reboot system, so is strongly recommeded run **fsck -p /dev/sda1** after system boots_


## Important tasks after installation ##
  * http://code.google.com/p/debian-for-alix/wiki/AttentionPurgeKnownKeys
  * http://code.google.com/p/debian-for-alix/wiki/AttentionChangeDefaultPasswords