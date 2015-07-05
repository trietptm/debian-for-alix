# Introduction #

For security reasons, is highly recommeded that you replace the default **ssh host keys** and **ssl certificates**.


# ssh hosts keys #

Anyone that download this image will know the ssh host keys content, doing steps below you'll create your unique ones

```
remountrw

rm /ro/etc/ssh/ssh_host_*
dpkg-reconfigure openssh-server
mv /rw/etc/ssh/ssh_host_* /ro/etc/ssh/

remountro

/etc/init.d/ssh restart
```

## Authority, Server and Clients SSL Certificates ##

As ssh hosts keys, the SSL certificates and keys are the same for anyone that get this prepared image, so to avoid broken communication and fake vpn clients (but with signed and valid certificates), you need to clear the current CA and create your new own.

```
remountrw

cd /ro/etc/ssl/easy-rsa/
. ./vars
./clean-all
./build-ca
./build-key-server crltest.site
./revoke-full crltest.site
./build-key-server alix.site
./build-dh
chmod 0755 keys
cd keys/
ln -s ca.crt $(openssl x509 -hash -in ca.crt -noout).0
ln -s crl.pem $(openssl x509 -hash -in ca.crt -noout).r0

remountro

/etc/init.d/nginx restart
/etc/init.d/stunnel4 restart
/etc/init.d/cups restart
/etc/init.d/openvpn restart
```