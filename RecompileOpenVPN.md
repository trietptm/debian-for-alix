#recompile openvpn with --iproute command option.

# Introduction #

Recompile openvpn with **--iproute** command option


# Details #
```
remountrw
rebind on
chroot /ro

cd /root
apt-get install liblzo2-2 liblzo2-dev
wget -S http://swupdate.openvpn.org/community/releases/openvpn-2.2.2.tar.gz
tar xvf openvpn-2.2.2.tar.gz
cd openvpn-2.2.2
autoreconf -vi
./configure --enable-iproute2
make
make install # or use checkinstall to generate and install a debian package

history -c ; exit
rebind off
remountro
```

  * If openvpn is installed, remove it before to avoid mistakes.
```
apt-get remove openvpn-blacklist openvpn
```

  * If you need a trivial (bridged or routed) setup, just install openvpn default package, provided by debian team
```
remountro
rebind on
chroot /ro

apt-get install openvpn
/etc/init.d/openvpn stop

history -c; exit

rebind off
remountro


```