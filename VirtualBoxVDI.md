#This wiki documents the steps taken to convert the Debian for Alix image to a VirtualBox vdi and extend the partition for testing purposes.

# VirtualBox Installation #


We will download and merge the Debian for Alix image and convert it to a VirtualBox image and extend the disk size to accommodate the installation of larger software packages.  The disk size will be doubled from 2GB to 4GB.  We will then have to modify Grub so the image will boot in VirtualBox.  We will need to first boot from a CD and make the modifications with VBoxManage (located in the VirtualBox install directory).

```
C:\>VBoxManage convertdd debian-for-alix-cf2g.img debian-for-alix-cf2g.vdi
C:\>VBoxManage modifyhd debian-for-alix-cf2g.vdi --resize 4000 
```

Download the Live disk of Debian Squeeze from http://www.debian.org/CD/live/

http://cdimage.debian.org/debian-cd/current-live/i386/iso-hybrid/debian-live-6.0.4-i386-gnome-desktop.iso


Create a new VBox virtual machine using debian-for-alix-cf2g.vdi as the hard disk. Add a CD/DVD device and select the Debian live CD image downloaded.  Boot the live CD, extend the partition using gparted, then mount the Alix image, modify /etc/default/grub to disable serial console, reinstall grub and reboot.  SIMPLE!  :-)

After booting the live CD open a Terminal window and install gparted.

```
$sudo -s
#dhclient eth0;apt-get update;apt-get install gparted
```
Use gparted to extend the partition to fill the space created by the VBoxManage command.  This will not affect the data.  Once complete we can then mount the disk and make the changes to grub.
```
#mount /mnt/sda1 /mnt
#mount --bind /dev /mnt/dev
#mount --bind /sys /mnt/sys
#mount --bind /proc /mnt/proc
#chroot /mnt

#pico /etc/default/grub
	#comment these lines 
	GRUB_CMDLINE_LINUX="aufs=tmpfs console=tty0 console=ttyS0,38400n8 reboot=bios"
	GRUB_TERMINAL=serial
	GRUB_SERIAL_COMMAND="serial --speed=38400"
	
#update-grub
#exit
#shutdown -h now
```
Shutdown the VM, remove the live CD from the settings, and boot to your Debian for Alix environment.