# howto configure a **wireless adapter** to connect alix on your wireless network

= install wpa\_supplicant package
```
remountrw
rebind on
chroot /ro

apt-get install wpasupplicant
history -c
exit

rebind off
remountro
```

# wpa\_supplicant configuration for boards with 3, 2 and 1 RJ45 connector #
_It was tested with an Atheros miniPCI **ath5k** and a Ralink USB dongle **rt73usb**, both accessing 802.11g and WPA2 over 2.4Ghz_

```
# /etc/wpa_supplicant/wpa_supplicant.conf
# configuration was generated using command below
# wpa_passphrase "SSID HERE" "PASSPHRASE HERE" > /etc/wpa_supplicant/wpa_supplicant.conf

network={
	ssid="SSID HERE"
	#psk="PASSPHRASE HERE"
	psk=a42608a7cc99982c1c7545a7ba891e678dd705f218b12537bbfc28f4d847ac5f
}
```

_Change parameters **SSID HERE**, and **PASSPHRASE HERE** according your environment_

# 3 RJ45 connectors #
## ALIX2D13 and ALIX2D3 ##
  * 1 WAN
  * 2 LAN + 1 WLAN
```
#/etc/network/interfaces
auto lo
iface lo inet loopback

allow-hotplug eth0
iface eth0 inet dhcp
    pre-up /etc/firewall/alix.fw start

auto tap0 br0
iface br0 inet static
    address 172.16.210.254
    netmask 255.255.255.0
    bridge_ports eth1 eth2 wlan0
    bridge_stp off
    bridge_maxwait 1

iface eth1 inet manual

iface eth2 inet manual

iface tap0 inet manual
    pre-up /usr/sbin/openvpn --dev tap0 --mktun

iface wlan0 inet manual
    pre-up wpa_supplicant -B -Dwext -i$IFACE -c/etc/wpa_supplicant/wpa_supplicant.conf
    post-down killall -q wpa_supplicant
```

# 2 RJ45 connectors #
## ALIX2D2 and ALIX6F2 ##
  * 1 WAN
  * 1 LAN + 1 WLAN
```
#/etc/network/interfaces
auto lo
iface lo inet loopback

allow-hotplug eth0
iface eth0 inet dhcp
    pre-up /etc/firewall/alix.fw start

auto tap0 br0
iface br0 inet static
    address 172.16.210.254
    netmask 255.255.255.0
    bridge_ports eth1 wlan0
    bridge_stp off
    bridge_maxwait 1

iface eth1 inet manual

iface tap0 inet manual
    pre-up /usr/sbin/openvpn --dev tap0 --mktun

iface wlan0 inet manual
    pre-up wpa_supplicant -B -Dwext -i$IFACE -c/etc/wpa_supplicant/wpa_supplicant.conf
    post-down killall -q wpa_supplicant
```

# 1 RJ45 connector #
## ALIX3D2 ##
  * 1 LAN + 1 WLAN
```
#/etc/network/interfaces
auto lo
iface lo inet loopback

auto tap0 br0
iface br0 inet static
    address 172.16.210.254
    netmask 255.255.255.0
    bridge_ports eth0 wlan0
    bridge_stp off
    bridge_maxwait 1

iface eth0 inet manual

iface tap0 inet manual
    pre-up /usr/sbin/openvpn --dev tap0 --mktun

iface wlan0 inet manual
    pre-up wpa_supplicant -B -Dwext -i$IFACE -c/etc/wpa_supplicant/wpa_supplicant.conf
    post-down killall -q wpa_supplicant
```

```
# use your preferred text editor, I've used nano
remountrw

wpa_passphrase "SSID HERE" "PASSPHRASE HERE" > /ro/etc/wpa_supplicant/wpa_supplicant.conf
nano /ro/etc/network/interfaces

remountro
```