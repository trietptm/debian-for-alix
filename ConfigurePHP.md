#install and configure php5 under nginx.

# Introduction #

How to install and configure php5 under nginx https server.
  * install php-cgi engine
  * create php-fcgi script and install it as service
  * create folder that will used to store php-fcgi socket
  * configure nginx to pass php requests to php-fcgi engine by socket
  * start php-fcgi engine
  * restart nginx http server

  * _any aditional php module can be installed, regarding hardware power limitations_
  * _**static** (htm, html, css, jpg, gif ...) and **php** files stored under **/var/www/htdocs**_
  * _**cgi** files stored under **/var/www/cgi-bin**_

# Details #

```
remountrw
rebind on
chroot /ro

apt-get install php5-cgi
mkdir /var/run/php
chown www-data /var/run/php

nano /etc/init.d/php-fcgi # see php-fcgi content below
chmod +x /etc/init.d/php-fcgi
insserv -d php-fcgi

nano /etc/nginx/conf.d/alix.conf # see alix.conf changed content below

apt-get clean
history -c
exit

rebind off
remountro

/etc/init.d/php-fcgi start
/etc/init.d/nginx restart
```

**alix.conf** configuration file
```
server {
    listen       [::]:80;
    server_name  $hostname;
    root         /mnt;


    auth_basic "Restrited Access";
    auth_basic_user_file /var/www/.htpasswd-storage;

    location / {
        autoindex on;
        autoindex_localtime on;
    }
}

server {
    listen       [::]:443;
    server_name  $hostname;
    root         /var/www/htdocs;

    ssl                  on;
    ssl_certificate      /etc/ssl/easy-rsa/keys/alix.site.crt;
    ssl_certificate_key  /etc/ssl/easy-rsa/keys/alix.site.key;

    ssl_session_timeout  5m;

    ssl_protocols  SSLv3 TLSv1;
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers   on;

    auth_basic "Restrited Access";
    auth_basic_user_file /var/www/.htpasswd;

    location / {
        index  index.html index.htm;
    }

    location /cgi-bin {
        gzip off;
        root /var/www;

        fastcgi_pass unix:/var/run/fcgiwrap.socket;
        include /etc/nginx/fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~ \.php$ {
        gzip off;

        fastcgi_pass unix:/var/run/php/php-fcgi.socket;
        include /etc/nginx/fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }


    location /rrd {
        add_header Pragma no-cache;
        expires -604800;
    }

    location /mnt {
        alias /mnt;
        autoindex on;
        autoindex_localtime on;
    }
}
```

**php-fcgi** service script
```
#!/bin/bash
### BEGIN INIT INFO
# Provides:          php-fcgi
# Required-Start:    $nginx
# Required-Stop:     $nginx
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts php over fcgi
# Description:       starts php over fcgi
### END INIT INFO

(( EUID )) && echo ‘You need to have root priviliges.’ && exit 1
BIND=/var/run/php/php-fcgi.socket
USER=www-data
PHP_FCGI_CHILDREN=5
PHP_FCGI_MAX_REQUESTS=1000

PHP_CGI=/usr/bin/php-cgi
PHP_CGI_NAME=`basename $PHP_CGI`
PHP_CGI_ARGS="- USER=$USER PATH=/usr/bin PHP_FCGI_CHILDREN=$PHP_FCGI_CHILDREN PHP_FCGI_MAX_REQUESTS=$PHP_FCGI_MAX_REQUESTS $PHP_CGI -b $BIND"
RETVAL=0

start() {
      echo -n "Starting PHP FastCGI: "
      start-stop-daemon --quiet --start --background --chuid "$USER" --exec /usr/bin/env -- $PHP_CGI_ARGS
      RETVAL=$?
      echo "$PHP_CGI_NAME."
}

stop() {
      echo -n "Stopping PHP FastCGI: "
      start-stop-daemon --quiet --stop --user $USER --exec $PHP_CGI
      RETVAL=$?
      echo "$PHP_CGI_NAME."
}

status() {
      PID="$(pidof $PHP_CGI_NAME)"
      [ "$PID" != "" ] && {
          echo "$PHP_CGI_NAME is running ($PID)"
      } || {
          echo "$PHP_CGI_NAME is not running"
      }
}


case "$1" in
    start)
      start
  ;;
    stop)
      stop
  ;;
    restart)
      stop
      start
  ;;
    status)
      status
  ;;
    *)
      echo "Usage: php-fcgi {start|stop|restart}"
      exit 1
  ;;
esac
exit $RETVAL
```