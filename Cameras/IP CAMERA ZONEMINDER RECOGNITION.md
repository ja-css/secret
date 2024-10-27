https://github.com/ZoneMinder/zmeventnotification  
https://github.com/ZoneMinder/mlapi  
  
???? https://github.com/pliablepixels/zmeventnotification  
  
https://zmeventnotification.readthedocs.io/en/latest/index.html  
http://www.onvif.org/specs/stream/ONVIF-Streaming-Spec-v210.pdf  
  
Neolink - RTSP for their native protocol (rust!)  
https://github.com/thirtythreeforty/neolink  
  
some tech info:  
https://wiki.zoneminder.com/Reolink  
update FW  
https://support.reolink.com/hc/en-us/articles/900004550323-How-to-Upgrade-Firmware-via-Reolink-Client-New-Client/  
reverse eng  
https://www.thirtythreeforty.net/posts/2020/05/hacking-reolink-cameras-for-fun-and-profit/  
  
#### Doorbell  
FW 2 way Audio via ONVIF: https://community.reolink.com/topic/5430/doorbell-first-new-firmware-support-more-smart-home-features-onvif-with-two-way-audio  
  
Java ONVIF https://github.com/RootSoft/ONVIF-Java  
Java ONVIF stream Player https://github.com/kingron/OnvifPlayer  
  
Note  
-----  
The master branch is always cutting edge. If you are packaging the ES into your own system/image it is recommended you use the [latest stable release](https://github.com/pliablepixels/zmeventnotification/releases/latest). See [this note](https://zmeventnotification.readthedocs.io/en/latest/guides/install.html#installation-of-the-event-server-es).  
  
  
What  
----  
The Event Notification Server sits along with ZoneMinder and offers real time notifications, support for push notifications as well as Machine Learning powered recognition.  
As of today, it supports:  
* detection of 80 types of objects (persons, cars, etc.)  
* face recognition  
* deep license plate recognition  
  
I will add more algorithms over time.  
  
Documentation  
-------------  
- Documentation, including installation, FAQ etc.are [here for the latest stable release](https://zmeventnotification.readthedocs.io/en/stable/) and [here for the master branch](https://zmeventnotification.readthedocs.io/en/latest/)  
- Always refer to the [Breaking Changes](https://zmeventnotification.readthedocs.io/en/latest/guides/breaking.html) document before you upgrade.  
  
3rd party dockers  
------------------  
I don't maintain docker images, so please don't ask me any questions about docker environments. There are others who maintain docker images.  
Please see [this link](https://zmeventnotification.readthedocs.io/en/latest/guides/install.html#rd-party-dockers)  
  
Requirements  
-------------  
- Python 3.6 or above  
  
Screenshots  
------------  
  
Click each image for larger versions. Some of these images are from other users who have granted permission for use  
  
<img src="https://github.com/pliablepixels/zmeventnotification/blob/master/screenshots/person_face.jpg?raw=true" width="300px" />   
<img src="https://github.com/pliablepixels/zmeventnotification/blob/master/screenshots/delivery.jpg?raw=true" width="300px" />   
<img src="https://github.com/pliablepixels/zmeventnotification/blob/master/screenshots/car.jpg?raw=true" width="300px" />   
<img src="https://github.com/pliablepixels/zmeventnotification/blob/master/screenshots/alpr.jpg?raw=true" width="300px" />