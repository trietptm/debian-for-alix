# configuration suggestions for files **/etc/hostapd/hostdapd.conf** and **/etc/network/interfaces**, when using a **wireless adapter** as an **access point**.

_dnsmasq (dns and dhcp server) serves LAN e WLAN through bridge interface br0_

# hostapd configuration for boards with 3, 2 and 1 RJ45 connector #
_It was tested with an Atheros miniPCI **ath5k** and a Ralink USB dongle **rt73usb**, both provided 802.11g and WPA2 over 2.4Ghz_

```
#/etc/hostapd/hostapd.conf
interface=wlan0
bridge=br0
ssid=ALIX-WIFI
driver=nl80211
channel=3
hw_mode=g
ignore_broadcast_ssid=0
auth_algs=1
macaddr_acl=0
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_passphrase=YOUR-PASS-KEY-HERE
wpa_pairwise=TKIP
rsn_pairwise=CCMP
wpa_group_rekey=600
ctrl_interface=/var/run/hostapd
```

_Change parameters **ssid**, and **wpa\_key\_mgmt** according your  preference_

_**hw\_mode** and **channel** perhaps need changes, see your hardware specs_

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
```

```
# use your preferred text editor, I've used nano
remountrw

nano /ro/etc/network/interfaces
nano /ro/etc/hostapd/hostapd.conf

remountro
```

_Enable service **hostapd** to start on **boot**_

```
remountrw
rebind on

chroot /ro
sysv-rc-conf hostapd on
history -c ; exit

rebind off
remountro
```