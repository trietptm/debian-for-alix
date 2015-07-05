# Client to Site VPN using iOS

**IPSec VPN Server** is largest used for secure communication. This wiki explains how to connect an iOS client (e.g., iPhone, iPad) to an Alix VPN server. The proposed solution does not use L2TP tunnel; therefore, it is suitable also for other clients that have native IPSec capabilities.

_This setup use PSK for group authentication._

_Clients are authenticated by means of system authentication (i.e., clients are users of the Alix server), but the softwares also support certificates or LDAP authentication._

_Tested clients: iOS 5.X, 6.x._

_In the proposed configuration, the VPN server assigns to each connected client an IP address in the range 192.168.2.20-192.168.2.29. It is possible to change the range and its size._

_$publicVPNaddress is the public IP address of the Alix VPN server._

_If the VPN server has only a private IP address (i.e., it is behind a gateway/firewall NAT), $publicVPNaddress is the public IP of the firewall. Obviously, in this case it should be natted with the private IP address of the VPN server, i.e., $local\_VPN\_IP\_address._

### Packages ###

  * racoon
  * some dependencies

### Config files ###

  * /etc/racoon/racoon.conf
  * /etc/racoon/psk.txt

```
remountrw
rebind on
chroot /ro
apt-get install racoon
/etc/init.d/racoon stop
exit
rebind off
remountro
```

/etc/racoon/racoon.conf
```
path pre_shared_key "/etc/racoon/psk.txt";

remote anonymous { 
 passive on; # Don't initiate, only listen
 exchange_mode main,aggressive; # Accept both modes
 my_identifier fqdn "<publicVPNaddress>"; # Identify ourselves with this name; change according to your public IP address or public DNS name
 mode_cfg on; # configure the client's IP address using mode configuration
# script "/etc/racoon/phase1-up.sh" phase1_up; # to be used if you need to execute scripts when the tunnel is established
verify_cert off; # Don't check client certificate
 ike_frag on; # Announce IKE-fragmentation support
 generate_policy on; # automatically install SPD's
 nat_traversal on; # Support NAT traversal
 dpd_delay 20; # Disconnect dead clients after 20 seconds
 proposal { # Phase 1 parameters
  encryption_algorithm aes;
  hash_algorithm sha1;
  authentication_method xauth_psk_server; # Require PreSharedKey group authentication and username/password user authentication 
  dh_group 2;
  }
}

mode_cfg {
 auth_source system; # Authenticate against system; change to LDAP if you use LDAP or change as you prefer accordin to your authentication method
 save_passwd on; # Allow users to save passwords

 group_source system; # Verify group membership in system

 network4 192.168.2.20;  # Give clients addresses starting from this address; change accordingly to your network settings
 pool_size 10;  # up to 10 addresses higher; change accordingly to your network settings
 dns4 <internal_DNS_IP_address>; # change accordingly to your network settings

 traffic to these subnets
 banner "/etc/racoon/msg.txt";   #change accordingly to your network settings if you want that a message will be explained after the connection; otherwise, comment this line
}

sainfo anonymous {
 encryption_algorithm aes;
 authentication_algorithm hmac_sha1;
 compression_algorithm deflate;
}

```

/etc/racoon/psk.txt
```
$your_VPN_group_name   $group_password
```


### allow traffic, add firewall rules ###

Add the lines below on firewall rules at **/ro/etc/firewall/alix.fw**
```
/sbin/iptables -A INPUT -p udp --dport 500  -m state --state NEW -j ACCEPT
/sbin/iptables -A INPUT -p udp --dport 4500 -m state --state NEW -j ACCEPT

/sbin/iptables -A FORWARD -s 192.168.2.0/24 -j ACCEPT
/sbin/iptables -A FORWARD -d 192.168.2.0/24 -j ACCEPT
```


- The last two firewall lines allow the traffic from the VPN clients toward the intranet.

- If the VPN server has only a private IP address (i.e., it is behind a gateway/firewall NAT), to allow the traffic from intranet clients and VPN clients (in the opposite direction), it is necessary to put accordingly a rule in the routing table of router (see configuration rules of your router). The rule should be something like: _send traffic to 192.168.2.0/24 through  Alix VPN server_.
For example, if you use a linux router/firewall, you should consider the rule _route add -net 192.168.2.0 netmask 255.255.255.0 gw $local\_VPN\_IP\_address._


# iPhone/iPad configuration #
- Configure a (Cisco) IPSec tunnel.

- The Description can be what you want.

- Server should be the FQDN of your server (e.g., $publicVPNaddress), this is checked against the my\_identifier field in the racoon config.

- Account and Password are the credentials of the Alix system user (e.g., login and pwd of an authorized user on the Alix VPN server).

- Group Name and Secret are the group credentials as specified in the
psk.txt file (e.g., $your\_VPN\_group\_name and $group\_password)