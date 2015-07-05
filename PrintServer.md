#Share your USB Printer through the CUPS.

# Setup your printer using CUPS Web Admin #

  * Plug printer device in an USB port
  * Open http://alix.site:631
  * Go to Administration section (root login will requested), click in Add Printer and follow steps
  * After configuration was successfully made, move configuration from volatile to persistent partition.

```
remountrw
mv /rw/etc/cups/* /ro/etc/cups/
remountro
```

_You won't need linux drivers on that page, you can use **raw driver** (Make: Raw and Model: Raw Queue), because printer client uses its own driver to gerenate data that will be sent to printer through CUPS, in this case, CUPS acts as gateway, forwarding the bytes from network client to USB device_

_Other network printers can be shared through CUPS, it provides advantage to put all queues in a central point_