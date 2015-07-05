# It's strongly recommended that you change the root password **123456** to another one

# change root password in a persistent way #
```
remountrw
rebind on

chroot /ro
passwd
history -c ; exit

rebind off
remountro
```


## Tip ##
_To get strong passwords, use pwgen. It can help you a lot!_
```
# example of usage
pwgen -c -n -s 12 1
```

# Access to other resources also can be changed #

> ## Simplified Web Admin Panel - https://alix.site/ ##

It´ll designed to most common operations, just a couple functions were available under login **admin** and password **123456**

Encrypt a new **secret**
```
perl -le 'print crypt("your-text-clear-password", "salt-hash")'
```

Get encrypted string and paste into file **/var/www/.htpasswd**
```
admin:sasQBhz6TYPA2
```

_You also can change the username **admin** to other one. Perhaps you wish to create another users_

_Don't forget to persist your changes moving files from **/rw** to **/ro**_

> ## Storage Access through browser - http://alix.site/ ##

By default that resource has no protection, to protect it follow steps below:

Encrypt a new **secret**
```
perl -le 'print crypt("new-guest-password", "salt-hash")'
```

Get encrypted string and paste into file **/var/www/.htpasswd-storage**
```
guest:sadZcJuXa3yRc
```

Edit **/etc/nginx/conf.d/alix.conf** and remove **#** from **auth\_basic** and **auth\_basic\_user\_file** inside first server block.
```
server {
    listen       80;
    server_name  $hostname;

    location / {
        root /mnt/;
        autoindex on;
        autoindex_localtime on;

        auth_basic "Restrited Access";
        auth_basic_user_file /var/www/.htpasswd-storage;
    }
}

server {
    listen       443;
    server_name  $hostname;
...
```

Restart nginx http server
```
/etc/init.d/nginx restart
```

_Don't forget to persist your changes moving files from **/rw** to **/ro**_

> ## Storage Access through CIFS/Samba/Win Share - \\alix\mnt ##

By default that resource has no protection, to protect it follow steps below:

Edit file **/etc/samba/smb.conf**, change line **security = SHARE** to **security = USER**

Default password for **guest** user is **123456**, change it running **smbpasswd -a nobody**

After that, restart samba service: **/etc/init.d/samba restart**

_If you intend to persist this, the changed files are **/etc/samba/smb.conf** and **/etc/samba/private/passdb.tdb**, move them from **/rw** to **/ro**_