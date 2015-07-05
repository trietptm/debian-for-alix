# Introduction #

Your debian-for-alix comes with the ath5k driver geared towards Atheros 5xxx chipsets. The version while functional for most needs however is old (0.6.0- experimental). Therefore it may be best to update these wireless drivers especially if you require more than just connecting to an access point, for example, getting noise and specific information about the channel currently being used.

Should you not be using and Atheros based chipset, or instead Ath6xxx or Ath9xxx based chipsets, the following will provide you with a choice of the latest drivers including ath5k.


# Download packages #

We will be installing the latest linux compat-wireless package available from

http://wireless.kernel.org/en/users/Download

Most latest releases have new patches and bug fixes, but also sometimes new bugs, so you have a choice of using a new daily release or one marked as stable.

You might also find you want to update the wireless tool iw, you cant get a latest package from

http://linuxwireless.org/download/iw/

## Installing ##

You can transfer both the compat package and the iw package using a CF-CARD reader or if you have you Alix board connected to the internet, you can use wget to download them using the links above.

Now bearing in mind we want the changes to persist after each boot, we are going to have to alter read only memory, execute the following commands:

```
remountrw
rebind on
chroot /ro
```

first purge the previous iw installation:

```
apt-get purge iw
```

Now update your system and install the following:

```
apt-get update
apt-get install libnl-dev ethtool
```

ethtool is used for profiling an interface, and libnl are the headers that the iw source needs to compile against. Also, unless you haven't already, make sure you install the package build-essential.

Now move into the /iw-3.x/ folder and type make:

```
cd /iw-3.x
make
```

then make install:

```
make install
```

If everything goes smoothly you will have updated the iw package. If you encounter a problem you may have to set an environment variable, see the readme of the iw package you installed for this.

Next we will build the drivers we want and install whichever suits the chipset type.

cd into the compat directory, and run the driver select script.

```
cd /compat-wireless-2012-09-18/
./scripts/driver-select atheros
```

If you simply run the script without specifying anything, it will return a set of options to you detailing all the drivers it currently supports. For our purposes we chose the atheros group of drivers.

Next type make, and wait as it builds the drivers. This can take some time, depending on what you have selected. After it has completed type make install.

```
make
make install
```

Lastly unload any drivers currently in use and use modprobe to select whichever driver you need, in our case it was ath5k.

```
make wlunload wireless modules
modprobe ath5k
```

Don't forget to return to read only memory, and then once you have you can reboot.

```
history -c ; exit
rebind off
remountro

shutdown -r -t secs 0
```

### Check ###

To check we have done everything correctly you can issue the following commands:

```
ethtool -i wlan0
modinfo ath5k  
```

modinfo gives the status of the driver currently being used. Specify any driver appropriate.
You can now begin using the iw tool and investigate your spectrum environment for example:

```
iw dev wlan0 survey dump

Survey data from wlan0
	frequency:			2412 MHz
	noise:				-95 dBm
	channel active time:		0 ms
	channel busy time:		0 ms
	channel receive time:		0 ms
	channel transmit time:		0 ms
```
