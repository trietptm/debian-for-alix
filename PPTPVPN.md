# Client to Site VPN


**PPTP VPN Server** was chose because MS Win has a built in client since XP, other OS like Mac OS, iOS, Android and Linux also have native support.

That service is enabled by default, to use it you just need to add users on **/etc/ppp/chap-secrets**

After client connects, It'll receive an IP in 172.16.210.201 - 172.16.210.220 range, so it'll able to access any other hosts in your local network as It was there.

![http://debian-for-alix.googlecode.com/files/openvpn-bridge.png](http://debian-for-alix.googlecode.com/files/openvpn-bridge.png)

I've used nano, but you can use your preferred text editor
```
remountrw

nano /ro/etc/ppp/chap-secrets

remountro
```

/ro/etc/ppp/chap-secrets example
```
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
john            pptpd   johnPWD                 *
mary            pptpd   maryPWD                 *
lucy            pptpd   lucyPWD                 *
peter           pptpd   peterPWD                *
```

### allow traffic, add firewall rules ###

Add the lines below on firewall rules at **/ro/etc/firewall/alix.fw**
```
/sbin/iptables -A INPUT -i ppp+ -j ACCEPT

/sbin/iptables -A INPUT -p tcp --dport 1723 -m state --state NEW -j ACCEPT
/sbin/iptables -A INPUT -p gre -j ACCEPT

/sbin/iptables -A FORWARD -i ppp+ -m state --state NEW -j ACCEPT
```