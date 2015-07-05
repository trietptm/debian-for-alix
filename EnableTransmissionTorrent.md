# Transmission #

To enable the well-known torrent software, **transmission**, just plug an external disk with a single partition NTFS, EXT2/EXT3 or FAT32`*` labeled P2P-HD.
When the **P2P-HD** is plugged for first time the structure below will be created:
```
transmission/
transmission/config
transmission/config/blocklists
transmission/config/resume
transmission/config/torrents
transmission/downloads
transmission/watch
```

Transmission Web Interface will turned available under  https://alix.site:9092
  * login: admin
  * pass: 123456

Transmission service will automatically **started** and **stopped** when **P2P-HD** will **plugged** and **unplugged**. You also use https://alix.site to manually start/stop service.

`*` FAT32 isn't recommended, because its max file size is limited to 4GB.

## Protect Web Interface Access ##

Change parameters on file **/mnt/P2P-HD/transmission/config/settings.json** as below:
```
    "rpc-authentication-required": true,
    "rpc-password": "YOU-PASSWORD-HERE",
    "rpc-username": "admin",
```
_You can access that file from Win PC through **Samba** sharing **\\alix\mnt\P2P-HD\transmission\config\settings.json**_

## Changing HD label and/or folder for torrent use ##

Edit **/etc/default/transmission-daemon** and do alter parameter **BASE\_DIR**

```
# /etc/default/transmission-daemon
ENABLE_DAEMON=0
BASE_DIR="/mnt/P2P-HD/transmission"
CONFIG_DIR="$BASE_DIR/config"
DOWNLOAD_DIR="$BASE_DIR/downloads"
WATCH_DIR="$BASE_DIR/watch"
OPTIONS="--config-dir $CONFIG_DIR --download-dir $DOWNLOAD_DIR --watch-dir $WATCH_DIR --no-incomplete-dir"
```

_Do not forget to persist change moving from /rw to /ro_