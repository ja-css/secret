  
  
To do the same with eth1 -> tunnel via eth0, the following should be required:    
  
### 1. Don't install hostapd, omit step 2.    
  
### 2. Replace wlan0 with eth1 everywhere    
  
### 3. Just in case, disable wifi    
`/boot/config.txt`  
  
Raspbian is managing hardware with overlays. In  `/boot/overlays/README` you will find:    
``` Name:   disable-wifi Info:   Disable onboard WLAN on Pi 3B, 3B+, 3A+, 4B and Zero W. Load:   dtoverlay=disable-wifi Params: <None> ```     
To disable WiFi, just add `dtoverlay=disable-wifi` to `/boot/config.txt` When disabled you will not get a WiFi interface **wlan0** as you can check with  `ip -br addr`.  
  
in /boot/config.txt  
`enable_uart=1`