`xinput list`  
find device id, let's say 7  
  
`xinput get-button-map {deviceId}`  
e.g.  
`xinput get-button-map {deviceId}`  
get button map, like 1 2 3 4 5 6 7 8 9 10  
  
set a new button map for a device id  
`xinput set-button-map {deviceId} {map}`  
  
e.g. to disable middle click  
`xinput set-button-map 7 1 0 2 3 4 5 6 7 8 9 10`  
  
or to have middle click behave like left click  
`xinput set-button-map 7 1 1 2 3 4 5 6 7 8 9 10`  
  
  
----  
  
Add to `rc.local`?  
  
  
ubuntu disable  bluetooth  
  
For Ubuntu 20.10 For this ubuntu edit /etc/bluetooth/main.conf