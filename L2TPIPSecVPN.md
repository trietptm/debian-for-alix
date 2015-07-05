# Client to Site VPN


**L2TP/IPSec VPN Server** is largest used for secure communication, it has native/built in clients for MS Win and other environments like Mac OS, iOS, Android and Linux.

_This setup use PSK, but the softwares also support certificates._

_Tested clients: Android 4.0, 4.1, iOS 6.0, Win 7 and 8_

After client connects, It'll receive an IP in 172.16.210.201 - 172.16.210.220 range, so it'll able to access any other hosts in your local network as It was there.

![http://debian-for-alix.googlecode.com/files/openvpn-bridge.png](http://debian-for-alix.googlecode.com/files/openvpn-bridge.png)

packages

  * xl2tpd
  * racoon
  * some dependencies

config files

  * /etc/ipsec-tools.conf
  * /etc/xl2tpd/xl2tpd.conf
  * /etc/racoon/racoon.conf
  * /etc/racoon/psk.txt
  * /etc/ppp/l2tp-options
  * /etc/sysctl.conf

```
remountrw
rebind on

chroot /ro

apt-get install xl2tpd racoon
/etc/init.d/racoon stop
/etc/init.d/xl2tpd stop
exit

rebind off
remountro
```

/etc/ipsec-tools.conf
```
#!/usr/sbin/setkey -f

flush;
spdflush;

spdadd 0.0.0.0/0[0] 0.0.0.0/0[1701] udp -P in  ipsec
        esp/transport//require;

spdadd 0.0.0.0/0[1701] 0.0.0.0/0[0] udp -P out ipsec
        esp/transport//require;
```

/etc/xl2tpd/xl2tpd.conf
```
[global]
force userspace = yes
ipsec saref = yes

[lns default]
ip range = 172.16.210.201-172.16.210.220
local ip = 10.254.254.254
hostname = alix.site
length bit = yes
pppoptfile = /etc/ppp/l2tp-options
```

/etc/racoon/racoon.conf
```
path include "/etc/racoon";
path pre_shared_key "/etc/racoon/psk.txt";
path certificate "/etc/racoon/certs";
path script "/etc/racoon/scripts";
log notify;
listen
{
        isakmp 172.16.210.254 [500];
        isakmp_natt 172.16.210.254 [4500];
        strict_address;
        adminsock disabled;
}
remote anonymous
{
        exchange_mode    main;
        passive          on;
        proposal_check   obey;
        support_proxy    on;
        nat_traversal    on;
        ike_frag         on;
        dpd_delay        20;
        proposal
        {
                encryption_algorithm  aes;
                hash_algorithm        sha1;
                authentication_method pre_shared_key;
                dh_group              modp1024;
        }
        proposal
        {
                encryption_algorithm  3des;
                hash_algorithm        sha1;
                authentication_method pre_shared_key;
                dh_group              modp1024;
        }
}
sainfo anonymous
{
        encryption_algorithm     aes,3des;
        authentication_algorithm hmac_sha1;
        compression_algorithm    deflate;
        pfs_group                modp1024;
}
```

/etc/racoon/psk.txt
```
# create your unique PSK secret
# generated using 'pwgen -s 30 1'

* FrzuZwmpDkqXVRvnFX1fxuCB7WsfYo
```

/etc/ppp/l2tp-options
```
name l2tpd
refuse-pap
refuse-chap
require-mschap-v2
auth
lock
crtscts
ms-dns 172.16.210.254
mtu 1400
mru 1400
nodefaultroute
noipv6
noipx
proxyarp
idle 1800
connect-delay 5000
lcp-echo-interval 30
lcp-echo-failure 4
#debug
#dump
```

/etc/sysctl.conf
```
# add (or uncomment) these lines, untouch others
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
```

Add users into chap-secrets, I've used nano, but you can use your preferred text editor
```
remountrw

nano /ro/etc/ppp/chap-secrets

remountro
```

/ro/etc/ppp/chap-secrets example
```
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
john            l2tpd   johnPWD                 *
mary            l2tpd   maryPWD                 *
lucy            l2tpd   lucyPWD                 *
peter           l2tpd   peterPWD                *
```

### allow traffic, add firewall rules ###

Add the lines below on firewall rules at **/ro/etc/firewall/alix.fw**
```
/sbin/iptables -A INPUT -i ppp+ -j ACCEPT

/sbin/iptables -A INPUT -p udp --dport 500  -m state --state NEW -j ACCEPT
/sbin/iptables -A INPUT -p udp --dport 4500 -m state --state NEW -j ACCEPT
/sbin/iptables -A INPUT -p udp --dport 1701  -m state --state NEW -j ACCEPT


/sbin/iptables -A FORWARD -i ppp+ -m state --state NEW -j ACCEPT
/sbin/iptables -A PREROUTING -i eth0 -p udp -m udp -m multiport --dports 500,4500 -j DNAT --to-destination 17.16.210.254 -t nat
```

# Tips #

I´ve experienced a long delay to start racoon service, the same occurred trying openswan, so seeing syslog messages and researching over the internet I found a not good solution, not loading Geode AES hardware accelerator driver.

/etc/modprobe.d/blacklist.conf
```
...
blacklist geode_aes
```

On this scenary, L2TP/IPSec server (ALIX) has a public IP, but if it has only a private IP, behind a gateway/firewall NAT, you´ll need to enable the resource below on Windows clients.

Windows-client-L2TP-IPsec-server-behind-NAT.reg
```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PolicyAgent]
"AssumeUDPEncapsulationContextOnSendRule"=dword:00000002
```