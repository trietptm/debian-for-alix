# OpenVPN in bridged mode #
Here you'll see how to use the pre-configured openvpn server to provide a **client to site** vpn in **bridged mode**.
  * **client to site** - when one or more single remote hosts can connect to a LAN
  * **bridge mode** - when the local and remote hosts act similar as they are in the same local network.

![http://debian-for-alix.googlecode.com/files/openvpn-bridge.png](http://debian-for-alix.googlecode.com/files/openvpn-bridge.png)

## SERVER ##
### turning bridge server on ###
```
remountrw
cd /ro/etc/openvpn/
mv udp-bridge.conf.OFF udp-bridge.conf
remountro
/etc/init.d/openvpn restart
```
add the line on firewall rules at /ro/etc/firewall/alix.fw
```
/sbin/iptables -A INPUT -p udp --dport 1194 -m state --state NEW -j ACCEPT
```

_bridged and routed can be used simultaneously, just change the port (1194) on one of them_
### creating client certificates ###
```
remountrw
cd /ro/etc/ssl/easy-rsa/
. ./vars
./build-key client1.site
./build-key client2.site
remountro
```

_client1.site.crt, client1.site.key, client2.site.crt and client2.site.key are created under /etc/ssl/easy-rsa/keys/ and they are used on clients_

## configuring Windows clients ##
### download and install openvpn ###
  * http://openvpn.net/index.php/download/community-downloads.html
  * http://swupdate.openvpn.org/community/releases/openvpn-2.2.2-install.exe

### config files and certificates ###
**Path for files on Win 7 64bits
```
C:\Program Files (x86)\OpenVPN\config\office-bridge.ovpn
C:\Program Files (x86)\OpenVPN\config\keys\ca.crt
C:\Program Files (x86)\OpenVPN\config\keys\client.crt
C:\Program Files (x86)\OpenVPN\config\keys\client.key
```**

**office-bridge.ovpn** content
```
client
dev tap
proto udp
remote host.yourdomain.com 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca   keys/ca.crt
cert keys/client.crt
key  keys/client.key
ns-cert-type server
comp-lzo
verb 3
```

**ca.crt**, **client.crt** and **client.key** content need to be copied from /etc/ssl/easy-rsa/keys/ on server.