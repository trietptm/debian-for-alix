# slightly different, only some directories are writeable in RAM disks

# Alternate Version #
## debian-for-alix-v2.img.bz2 ##

On this alternate version, the root partition stored on CF-CARD is mounted in readonly, while a couple others that write is required to system runs are allocated in RAM.

I think the maintenance of this version is a little bit simple than previous one.

Also, it is possible a better control of partitions in RAM by setting maximum amount to each one.

# Details #

All of system is under ro mode, except these:
```
/mnt/        (tmpfs)
/tmp/        (tmpfs)
/var/lock/   (aufs, union from /ro/var/lock  and /rw/var/lock )
/var/log/    (aufs, union from /ro/var/log   and /rw/var/log  )
/var/run/    (aufs, union from /ro/var/run   and /rw/var/run  )
/var/spool/  (aufs, union from /ro/var/spool and /rw/var/spool)
```

The size of each partition can be ajusted by setting variables **TMPFS\_SIZE**, **RUNFS\_SIZE**, **LOCKFS\_SIZE**, **LOGFS\_SIZE** and **SPOOLFS\_SIZE** in **/etc/default/tmpfs**

```
root@alix:~# df -hT
Filesystem    Type    Size  Used Avail Use% Mounted on
rootfs      rootfs    1.8G  864M  816M  52% /
none      devtmpfs    121M  104K  120M   1% /dev
/dev/sda1     ext2    1.8G  864M  816M  52% /
tmpfs        tmpfs     20M     0   20M   0% /lib/init/rw
tmpfs        tmpfs    256K     0  256K   0% /mnt
tmpfs        tmpfs     20M   24K   20M   1% /tmp
tmpfslock    tmpfs    5.0M   28K  5.0M   1% /rw/var/lock
tmpfslog     tmpfs     20M  280K   20M   2% /rw/var/log
tmpfsrun     tmpfs    5.0M  648K  4.4M  13% /rw/var/run
tmpfsspool   tmpfs     25M  2.5M   23M  10% /rw/var/spool
aufslock      aufs    5.0M   28K  5.0M   1% /var/lock
aufslog       aufs     20M  280K   20M   2% /var/log
aufsrun       aufs    5.0M  648K  4.4M  13% /var/run
aufsspool     aufs     25M  2.5M   23M  10% /var/spool
tmpfs        tmpfs     30M  4.0K   30M   1% /dev/shm
```


### Tips ###
**General maintenance in readonly system.**

To install some package or change some configuration file, use **remountrw** to turn **/** writable and **remountro** to back it to readonly mode.

If for some reason it needs to boot in readwrite mode, edit **/etc/fstab**
and change mount parameter from **ro,noatime...** to **rw,noatime...**
```
UUID=438cb640-b8ae-41c7-befc-6c99c476c9ba /  ext2 ro,noatime,errors=continue,acl 0 0
```
_don´t forget to change it back to **ro** mode after maintenance (or test) was finished._

**Speed up boot almost a half time**

turn root file system writable and start udev-mtab just once to collect and update informations about hardware.
```
/etc/init.d/udev-mtab start
```
_boot time compared on ALIX2D13_


### Security ###

Change root password
```
root@alix:/# passwd
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```

Remove ssh host keys and create new ones
```
root@alix:/# rm -v /etc/ssh/ssh_host_*
removed '/etc/ssh/ssh_host_dsa_key'
removed '/etc/ssh/ssh_host_dsa_key.pub'
removed '/etc/ssh/ssh_host_rsa_key'
removed '/etc/ssh/ssh_host_rsa_key.pub'

root@alix:/# dpkg-reconfigure openssh-server
Creating SSH2 RSA key; this may take some time ...
Creating SSH2 DSA key; this may take some time ...
Restarting OpenBSD Secure Shell server: sshd.
```

Build new certificate authority and certificates, they are used by openvpn, nginx, cups and stunnel/transmission to provide SSL/TLS communication.
```
root@alix:~# cd /etc/ssl/easy-rsa

root@alix:/etc/ssl/easy-rsa# . vars
NOTE: If you run ./clean-all, I will be doing a rm -rf on /etc/ssl/easy-rsa/keys

root@alix:/etc/ssl/easy-rsa# ./clean-all

root@alix:/etc/ssl/easy-rsa# ./build-ca
Generating a 1024 bit RSA private key
.......++++++
...++++++
writing new private key to 'ca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [BR]:
State or Province Name (full name) [SP]:
Locality Name (eg, city) [Sao Paulo]:
Organization Name (eg, company) [Debian For Alix]:
Organizational Unit Name (eg, section) []:Linux IT
Common Name (eg, your name or your server s hostname) [Debian For Alix CA]:
Name []:
Email Address [alix@site]:

root@alix:/etc/ssl/easy-rsa# ./build-dh
Generating DH parameters, 1024 bit long safe prime, generator 2
This is going to take a long time
..........+...........+..................................................................+....................................+....+....................+..................+...................................................+.................................................................................................................+.+................................+....+.................................++*++*++*

root@alix:/etc/ssl/easy-rsa# ./build-key-server sometorevoke.site
Generating a 1024 bit RSA private key
...........++++++
.......................................................++++++
writing new private key to 'sometorevoke.site.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [BR]:
State or Province Name (full name) [SP]:
Locality Name (eg, city) [Sao Paulo]:
Organization Name (eg, company) [Debian For Alix]:
Organizational Unit Name (eg, section) []:Linux IT
Common Name (eg, your name or your server s hostname) [sometorevoke.site]:
Name []:
Email Address [alix@site]:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Using configuration from /etc/ssl/easy-rsa/openssl.cnf
Check that the request matches the signature
Signature ok
The Subject s Distinguished Name is as follows
countryName           :PRINTABLE:'BR'
stateOrProvinceName   :PRINTABLE:'SP'
localityName          :PRINTABLE:'Sao Paulo'
organizationName      :PRINTABLE:'Debian For Alix'
organizationalUnitName:PRINTABLE:'Linux IT'
commonName            :PRINTABLE:'sometorevoke.site'
emailAddress          :IA5STRING:'alix@site'
Certificate is to be certified until Nov  6 02:27:26 2022 GMT (3650 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated

root@alix:/etc/ssl/easy-rsa# ./revoke-full sometorevoke.site
Using configuration from /etc/ssl/easy-rsa/openssl.cnf
Revoking Certificate 01.
Data Base Updated
Using configuration from /etc/ssl/easy-rsa/openssl.cnf
sometorevoke.site.crt: /C=BR/ST=SP/L=Sao Paulo/O=Debian For Alix/OU=Linux IT/CN=sometorevoke.site/emailAddress=alix@site
error 23 at 0 depth lookup:certificate revoked
root@alix:/etc/ssl/easy-rsa# ./build-key-server alix.site
Generating a 1024 bit RSA private key
........................++++++
................++++++
writing new private key to 'alix.site.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [BR]:
State or Province Name (full name) [SP]:
Locality Name (eg, city) [Sao Paulo]:
Organization Name (eg, company) [Debian For Alix]:
Organizational Unit Name (eg, section) []:Linux IT
Common Name (eg, your name or your server s hostname) [alix.site]:
Name []:
Email Address [alix@site]:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Using configuration from /etc/ssl/easy-rsa/openssl.cnf
Check that the request matches the signature
Signature ok
The Subject s Distinguished Name is as follows
countryName           :PRINTABLE:'BR'
stateOrProvinceName   :PRINTABLE:'SP'
localityName          :PRINTABLE:'Sao Paulo'
organizationName      :PRINTABLE:'Debian For Alix'
organizationalUnitName:PRINTABLE:'Linux IT'
commonName            :PRINTABLE:'alix.site'
emailAddress          :IA5STRING:'alix@site'
Certificate is to be certified until Nov  6 02:28:09 2022 GMT (3650 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated

root@alix:/etc/ssl/easy-rsa# chmod 0755 keys

root@alix:/etc/ssl/easy-rsa# /etc/init.d/openvpn restart
Stopping virtual private network daemon: udp-bridge.
Starting virtual private network daemon: udp-bridge.

root@alix:/etc/ssl/easy-rsa# /etc/init.d/nginx restart
Restarting nginx: nginx.

root@alix:/etc/ssl/easy-rsa# /etc/init.d/stunnel4 restart
Restarting SSL tunnels: [stopped: /etc/stunnel/stunnel.conf] [Started: /etc/stunnel/stunnel.conf] stunnel.

root@alix:/etc/ssl/easy-rsa# /etc/init.d/cups restart
Restarting Common Unix Printing System: cupsd.
```

Change samba `guest` password
```
root@alix:/# smbpasswd nobody
New SMB password:
Retype new SMB password:

root@alix:/# mv /rw/var/spool/samba/private/passdb.tdb /ro/var/spool/samba/private/
```

Change password for admin panel and storage web access. Optionally, also change usernames.
```
root@alix:/# perl -le 'print crypt("new-plain-pwd", "*/")'
*/fGLOb58BZ1Q

root@alix:/# nano /var/www/.htpasswd

root@alix:/# nano /var/www/.htpasswd-storage
```