#NOW UPDATED FOR V3 (Debian 7) - This wiki documents the steps taken to convert the Debian for Alix image to a VirtualBox vdi and extend the partition for testing purposes.

# VirtualBox Installation #


We will download and merge the Debian for Alix image and convert it to a VirtualBox image and extend the disk size to accommodate the installation of larger software packages.  The disk size will be doubled from 2GB to 4GB. We will need to first make the modifications with VBoxManage (located in the VirtualBox install directory).

```
C:\Users\edge>"..\..\Program Files\7-Zip\7z.exe" e Downloads\debian-for-alix-v3.img.bz2 -oDownloads

7-Zip [64] 9.20  Copyright (c) 1999-2010 Igor Pavlov  2010-11-18

Processing archive: Downloads\debian-for-alix-v3.img.bz2

Extracting  debian-for-alix-v3.img

Everything is Ok

Size:       1879048192
Compressed: 264929455

C:\Users\edge>"..\..\Program Files\Oracle\VirtualBox\VBoxManage.exe" convertdd Downloads\debian-for-alix-v3.img "VirtualBox VMs\debian-for-alix-v3.vdi"
Converting from raw image file="Downloads\debian-for-alix-v3.img" to file="VirtualBox VMs\debian-for-alix-v3.vdi"...
Creating dynamic image with size 1879048192 bytes (1792MB)...

C:\Users\edge>"..\..\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyhd "VirtualBox VMs\debian-for-alix-v3.vdi" --resize 4096
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
```

Download the Live disk of Debian Wheezy from http://www.debian.org/CD/live/

http://cdimage.debian.org/debian-cd/current-live/i386/iso-hybrid/debian-live-7.6.0-i386-gnome-desktop.iso


Create a new VBox virtual machine using debian-for-alix-v3.vdi as the hard disk. Add a CD/DVD device and select the Debian live CD image downloaded.  Boot the live CD, extend the partition using gparted.

After booting the live CD open a Terminal window and install gparted.

```
$sudo -s
#dhclient eth0;apt-get update;apt-get install gparted
```
Use gparted to extend the partition to fill the space created by the VBoxManage command.  This will not affect the data.  Shutdown the VM, remove the live CD from the settings, and boot to your Debian for Alix environment.

If you need step-by-step commands we will now use the command line tool fdisk and resize2fs and use them to extend the partition instead.  First boot to the Debian ISO disk and open a terminal window.

```
root@debian:~# fdisk /dev/sda

Command (m for help): u
Changing display/entry units to cylinders (DEPRECATED!)

Command (m for help): d
Selected partition 1

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
Using default value 1
First cylinder (1-2273, default 1):
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-2273, default 2273):
Using default value 2273

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

Reboot the virtual machine, remove the Live disk, and boot into Debian-for-Alix.  Login and run the resize2fs command to

```
root@alix:/home/alix# resize2fs /dev/sda1
resize2fs 1.42.5 (29-Jul-2012)
Filesystem at /dev/sda1 is mounted on /; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
Performing an on-line resize of /dev/sda1 to 1048165 (4k) blocks.
The filesystem on /dev/sda1 is now 1048165 blocks long.
```