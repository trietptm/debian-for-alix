# configuration suggestions for file **/etc/network/interfaces**

# 3 RJ45 connectors #
## ALIX2D13 and ALIX2D3 ##
  * 1 WAN
  * 2 LAN
```
auto lo
iface lo inet loopback

allow-hotplug eth0
iface eth0 inet dhcp
    pre-up /etc/firewall/alix.fw start

auto tap0 br0
iface br0 inet static
    address 172.16.210.254
    netmask 255.255.255.0
    bridge_ports eth1 eth2
    bridge_stp off
    bridge_maxwait 1

iface eth1 inet manual

iface eth2 inet manual

iface tap0 inet manual
    pre-up /usr/sbin/openvpn --dev tap0 --mktun
```

# 2 RJ45 connectors #
## ALIX2D2 and ALIX6F2 ##
  * 1 WAN
  * 1 LAN
```
auto lo
iface lo inet loopback

allow-hotplug eth0
iface eth0 inet dhcp
    pre-up /etc/firewall/alix.fw start

auto tap0 br0
iface br0 inet static
    address 172.16.210.254
    netmask 255.255.255.0
    bridge_ports eth1
    bridge_stp off
    bridge_maxwait 1

iface eth1 inet manual

iface tap0 inet manual
    pre-up /usr/sbin/openvpn --dev tap0 --mktun
```

# 1 RJ45 connector #
## ALIX3D2 ##
  * 1 LAN
```
auto lo
iface lo inet loopback

auto tap0 br0
iface br0 inet static
    address 172.16.210.254
    netmask 255.255.255.0
    bridge_ports eth0
    bridge_stp off
    bridge_maxwait 1

iface eth0 inet manual

iface tap0 inet manual
    pre-up /usr/sbin/openvpn --dev tap0 --mktun
```

Do not forget to persist the changes, otherwise will be lost on next boot

```
#alter file using you preferred editor
nano /etc/network/interfaces
remountrw
mv /rw/etc/network/interfaces /ro/etc/network/
remountro
```

_If you won't plan to use openvpn udp-bridge, you can remove or comment tap0 interface references_