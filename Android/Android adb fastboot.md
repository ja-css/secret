## Android adb fastboot on linux  
  
ADB - Android Debug Bridge  
  
`adb devices` - show devices list  
  
`sudo apt-get install android-tools-adb` - This will install the old version  
  
## press About phone -> Build number 10 times to unlock developer mode  
System -> Developer options  
    USB debugging  
    OEM unlock  
  
To show in adb:  
    Connected Devlices -> USB -> Use USB for File Transfer  
  
## Get image and flasher from   
https://calyxos.org/install/devices/bramble/linux/#download-factory-image  
  
## Get stock image from  
https://developers.google.com/android/images  
  
  
---  
  
  
## How to enter fastboot?  
  
### Method 1  
  
Power off the Android device.  
Hold the volume down button.  
Connect a USB cable to the host computer.  
  
The device should automatically power on into fastboot mode.  
  
### Method 2  
  
Power off the Android device.  
Hold the volume down button.  
Hold the power button.  
Wait for a slight vibration.  
Release the power button.  
When the fastboot screen appears, release the volume down button.  
  
---  
  
## How to use fastboot  
For the purpose of installing CalyxOS, you don’t need to do anything on the fastboot screen.  
  
If you do want to select one of the options, use the volume buttons to change the selected action, and the power button to trigger the action.  
  
  
  
## if `adb devices` shows "no permissions"  
A temporary fix would be to set the phone to File Transfer mode or MTP mode.  
  
## from fastboot  
  
`fastboot flashing unlock`  
  
  
## then  
  
run `device-flasher`  
  
## In this order  
  
timezone - keep default?  
  
turn off google fi (sim)  
  
network & internet   
    - connectivity check, turn off  
    - private dns cloudflare  
  
Developer Options ->  
    - disable OEM unlock  
    - disable automatics system updates  
  
Firewall - disable everytheng  
  
Disable apps, like Update Client  
    - keep Gallery to enable wallpapers  
  
DONT USE POTENTIALLY BAD FLASH  
copy/install apks  
  
  
Network and Internet - set up   
 global VPN   
 block non-vpn connections  
  
  
---  
  
## Back to stock android: here  
  
https://developers.google.com/android/images  
  
  
## Option 2 (using factory images): If you flashed your device with the factory image after unlocking the bootloader, after a successful boot into Android 13 for the first time  
  
- Extract the contents of the factory ROM .zip file, identify the bootloader image in the extracted files, and follow the sequence of events as listed below to flash the bootloader to both the slots. Substitute the name of the bootloader image with that of your device for the Pixel 6 and Pixel 6a.  
  
- Start the device in fastboot mode with one of the following methods:  
  
- Using the adb tool: With the device powered on, execute:  
  
```  
adb reboot bootloader  
```  
  
Using a key combo: Turn the device off, then turn it on and immediately hold down the relevant key combination for your device.  
  
- Flash the Android 13 bootloader to the inactive slot. The following command is specific to a particular build of a Pixel 6 Pro device. Substitute the name of the bootloader image determined in the first step above, if different, for the image file name argument.  
  
`fastboot` should be run as root  
  
Unlock flashing  
```  
fastboot flashing unlock  
```  
Flash bootloader  
```  
fastboot --slot=other flash bootloader bootloader-raven-slider-1.2-8739948.img  
```  
  
**STOPPED HERE**  
      
If the flash was successful this command will print OKAY [ ... ]. Do not proceed unless this command completes successfully.  
  
- After flashing the inactive slot bootloader to an Android 13 bootloader, reboot to that slot to ensure that the bootloader will be marked as bootable. Important: Please run the exact sequence of commands as listed below. Don't forget to enter the full line fastboot reboot bootloader when rebooting. Failure to do so may leave your device in an unbootable state.  
```  
    fastboot set_active other  
    fastboot reboot bootloader  
    fastboot set_active other  
    fastboot reboot bootloader  
  
```  
  
Reboot into the current OS. Either press the power button to boot or use the following fastboot command:  
```  
	fastboot reboot  
```  
  
**CONTINUED HERE**  
      
  
## Here's the OS flash  
### Flashing instructions  
The factory image downloaded from this page includes a script that flashes the device, typically named flash-all.sh (On Windows systems, use flash-all.bat instead).  
  
lock the bootloader  
from fastboot  
  
```  
fastboot flashing lock  
```  
  
Android can't do yubikey, choose - log in with security code, to go to g.co/sc