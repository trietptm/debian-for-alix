#This wiki will document the conversion of the Debian for Alix image to an image that can be used for wireless security assessments.

# Introduction #

You will need to have Debian for Alix installed on a Compact Flash card or converted to a VirtualBox vdi.


# Installing and Removing Software #

Once the Alix boots, whether on the Alix or in VirtualBox, run the following commands.

## Remove Software ##
```
#apt-get update
#apt-get remove cups samba vsftpd minidlna tinyproxy pptpd snmpd openvpn openconnect nginx
#apt-get autoremove
#insserv -r dnsmasq 
#insserv -r stunnel4
```
We remove dnsmasq from start since we will be using it in penetration testing.

## Install Software ##

http://code.google.com/p/debian-for-alix/wiki/PentestSoftwareInstallation?ts=1352063824&updated=PentestSoftwareInstallation

### Wireless Drivers for Packet Injection ###

Atheros AR5212 (ath5k and madwifi-ng)
Realtek RTL8187L (rtl8187)

Compile and Install Wireless Drivers to support Packet Injection (the drivers already support Monitor mode).  You will need to compile the trunk version of madwifi-ng driver for the Atheros chipset.  This is for the mini-PCI card installed in the Alix.

For more info see - http://madwifi-project.org/wiki/UserDocs/GettingMadwifi
```
#cd ~/source
root@alix:~/source# svn checkout http://madwifi-project.org/svn/madwifi/trunk madwifi
root@alix:~/source/madwifi# cd madwifi
root@alix:~/source/madwifi# make
root@alix:~/source/madwifi# make install
```

For our Alfa USB Wireless Adapters (AWUS036H and AWUS036EW) we will need to patch the rtl8187 drivers to support Packet Injection (see http://www.aircrack-ng.org/doku.php?id=r8187)

```
#ifconfig wlan0 down	 
#rmmod r8187 rtl8187 2>/dev/null
#cd ~/source
root@alix:~/source# wget http://dl.aircrack-ng.org/drivers/rtl8187_linux_26.1010.zip
root@alix:~/source# unzip rtl8187_linux_26.1010.zip
root@alix:~/source# cd rtl8187_linux_26.1010.0622.2006/
root@alix:~/source/rtl8187_linux_26.1010.0622.2006# wget http://patches.aircrack-ng.org/rtl8187_2.6.27.patch
root@alix:~/source/rtl8187_linux_26.1010.0622.2006# wget http://patches.aircrack-ng.org/rtl8187_2.6.32.patch
root@alix:~/source/rtl8187_linux_26.1010.0622.2006# tar xzf drv.tar.gz
root@alix:~/source/rtl8187_linux_26.1010.0622.2006# tar xzf stack.tar.gz
root@alix:~/source/rtl8187_linux_26.1010.0622.2006# patch -Np1 -i rtl8187_2.6.27.patch
root@alix:~/source/rtl8187_linux_26.1010.0622.2006# patch -Np1 -i rtl8187_2.6.32.patch
root@alix:~/source/rtl8187_linux_26.1010.0622.2006# make
root@alix:~/source/rtl8187_linux_26.1010.0622.2006# make install
```

## miniPCIe GSM Cellular Modem ##
_(Alix 6f2 Only)_

These instructions are for the SierraWireless (HP) MC8775 and may need to be tailored for your GSM card.  If the card is not unlocked this can be done from Windows with the software DC - Unlocker 2 (Client 1.00.0857).  The software wvdial is used to make the connection.
```
#apt-get install wvdial ppp
#vim /etc/wvdial.conf
```
**-----------------wvdial.conf-----------------**
```
[Dialer Defaults]
Phone = *99#
Stupid Mode = 1
Init1 = ATZ
Init2 = ATQ0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
Init3 = AT+CGDCONT=1,"IP","<your.provider.address>"
Modem Type = USB Modem
Baud = 115200
New PPPD = yes
Modem = /dev/ttyUSB0
ISDN = 0
Dial Command = ATDT
username = " "
password = " "
```

```
#vim /etc/network/interfaces 

auto ppp0
iface ppp0 inet wvdial
```

A great service to use with this device for penetration testing is T-Mobile's Prepaid Broadband No Contract service (1 day, 1 week, and 1 month).  Only activate it when you need it.  http://www.t-mobile.com/shop/Phones/cell-phone-detail.aspx?class=prepaid&cell-phone=Mobile-Broadband-SIM-ONLY-KIT-Prepaid

NOTE:  as of this document you cannot activate your miniPCIe card with your SIM on a Linux machine.  It has to be done with Windows.  If you do not have a laptop or desktop with a miniPCIe slot you will need to get an adapter.  Thankfully they make a USB adapter (for laptops) and PCI adapters for desktops.  The software DC-Unlocker is great at unlocking the card.  For the Serria Wireless AirCard Watcher is used to connect to the Internet
http://www.sierrawireless.com/productsandservices/AirCard/SoftwareSolutions/AirCardWatcher_USBs.aspx

## Reverse SSH Tunnel ##

Out of band access to the Alix 6f2 using the GSM cellular modem.

Manually setup the connection, including public/private keys.
Further help can be obtained from http://wiki.maemo.org/Reverse_ssh and  http://www.howtoforge.com/reverse-ssh-tunneling
SSH Public/Private Key help: http://newport.eecs.uci.edu/~chou/ssh-key.html

NOTE:  for this to work you will need a publicly accessible SSH server that you control.

**Simple**
Start Tunnel from Alix:
```
ssh -R 65001:localhost:22 username@remote.host.com
```
From Remote Host:
```
ssh root@localhost -p 65001
```

**With public/private keys**

For these instructions the Alix must be on the same network as the Remote Host (just for the key exchange), the key will be called alix\_key, and no password will be assigned to the key.  No password is assigned so that automatic reconnect of the tunnel can be done via a script when the connection goes down.  Also, these steps will not overwrite any current private key that was created (id\_rsa).  New keys are generated that we will only use for the reverse tunnel.

On Remote Host
```
$ ssh-keygen -t rsa -b 4048
Generating public/private rsa key pair.
Enter file in which to save the key (/home/edge/.ssh/id_rsa): /home/edge/.ssh/pentest_key
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/edge/.ssh/pentest_key.
Your public key has been saved in /home/edge/.ssh/pentest_key.pub.
```
On Alix (as root)
```
#mkdir ~/.ssh
#chmod 700 ~/.ssh
```
On Remote Host
```
$scp ~/.ssh/pentest_key.pub root@<alix-ip>:/root/.ssh/pentest_key.pub
```
On Alix
```
#cat ~/.ssh/pentest_key.pub >> ~/.ssh/authorized_keys
#chmod 600 ~/.ssh/authorized_keys
```
Now do the same steps but reverse Host and Alix

On Alix
```
#ssh-keygen -t rsa -b 4048  
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/alix_key
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/alix_key.
Your public key has been saved in /root/.ssh/alix_key.pub.
```
On Remote Host (as user)
```
$mkdir ~/.ssh
$chmod 700 ~/.ssh
```
On Alix
```
#scp ~/.ssh/alix_key.pub <username>@<remotehost-ip>:~/.ssh/alix_key.pub
```
On Remote Host
```
$cat ~/.ssh/alix_key.pub >> ~/.ssh/authorized_keys
$chmod 600 ~/.ssh/authorized_keys
```
Start Tunnel from Alix:
```
ssh -i ~/.ssh/alix_key -R 65001:localhost:22 username@remote.host.com
```
From Remote Host:
```
ssh -i ~/.ssh/pentest_key root@localhost -p 65001
```
**Persistent SSH**
Keep SSH connection open when dropped
```
#vim /etc/ssh/ssh_config

#Insert the following lines into the file
ExitOnForwardFailure yes
ServerAliveInterval 60
```
```
#while true; do ssh -I ~/.ssh/alix_key -N -C -p 22 -R 65001:127.0.0.1:22 username@remote.host.com; sleep 5; done
```
**Automated Persistent SSH with Autossh**
```
#cd ~/source
#wget http://www.harding.motd.ca/autossh/autossh-1.4c.tgz
#tar zxf autossh-1.4c.tgz
#cd autossh-1.4c
#./configure
#make
#make install
```
See /usr/local/share/examples/autossh/autossh.host for an example of an autossh start script.