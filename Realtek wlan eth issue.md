
firmware
https://linuxhint.com/update-raspberry-pi-firmware

---

wlan power
https://github.com/raspberrypi/linux/issues/1342

---

dmesg will show the device:
e.g.
```


----------


r8152 1-1:1.0 eth0: carrier off
r8152 1-1:1.0 eth0: carrier on
```
---------------------------------------------------------------

from
https://forum.endeavouros.com/t/ethernet-keeps-deactivating-and-reactivating-randomly/18851/58?page=3

also https://forums.raspberrypi.com/viewtopic.php?t=242964
and https://superuser.com/questions/1272259/debian-stretch-and-realtek-r8152-chipset-carrier-off-on-error

---------------------------------------------------------------

Ding ding ding, we have a winner, thank you so much!!.  
`sudo echo on | sudo tee /sys/bus/usb/devices/4-1/power/control`  

Does the trick… Now I only need to get it running on boot, but the script and service I made just won’t run. I made the following script:  
poweron.sh:

```bash
#!/bin/sh

echo on > /sys/bus/usb/devices/4-1/power/control 
exit 0

#"$@"

```

I did a  `sudo chmod +x poweron.sh`  to make it executable.  
The service that goes along with it “power-usb.service”

```ini
[Unit]
Description=ethernetpower

[Service]
ExecStart=/usr/bin/usbpower/power.sh

[Install]
WantedBy=multi-user.target
```

And I enabled and started the service with systemctl. But it still won’t run on startup.

Edit: The script was run to early in boot, all I needed to do was add a little sleep to the script:

```bash
#!/bin/sh
sleep 10
echo on > /sys/bus/usb/devices/4-1/power/control 
exit 0

#"$@"
```

-------------------------------------------

[pebcak]
Try this and see if it works:

```ini
[Unit]
Description=ethernetpower

[Service]
Type=simple
ExecStart=/usr/bin/usbpower/power.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

`sudo systemctl enable power-usb.service`