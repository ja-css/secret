https://emulationstation.org/gettingstarted.html#install_rpi_retropie  
  
## Installing on Raspberry Pi (Stand-alone)  
  
This is a guide for everything you need to install EmulationStation on a fresh Raspbian Stretch install. All the dependencies are in the Raspbian apt repositories.  
  
    
  
**Make sure everything is up to date**  
  
```  
sudo apt-get update  
sudo apt-get upgrade  
sudo rpi-update  
  
```  
  
    
  
**Set the minimum amount of RAM to the GPU**  
  
```  
sudo nano /boot/config.txt  
# add or replace "gpu_mem = 32"  
# if you skip this step, you will probably get "out of memory" errors when compiling  
```  
  
    
  
**Reboot to apply GPU RAM changes and make sure you're using the newest firmware**  
  
```  
sudo reboot  
```  
  
    
  
**Install dependencies for EmulationStation**  
  
```  
sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-date-time-dev libboost-locale-dev libfreeimage-dev libfreetype6-dev libeigen3-dev libcurl4-openssl-dev libasound2-dev cmake libsdl2-dev  
  
```  
  
    
  
**Compile and install EmulationStation**  
  
```  
git clone https://github.com/Aloshi/EmulationStation  
cd EmulationStation  
mkdir build  
cd build  
  
# On the RPi 2, you may need to add '-DFREETYPE_INCLUDE_DIRS=/usr/include/freetype2/'.  
# See issue #384 on GitHub for details.  
cmake ..  
  
# you can add -j2 here to use 2 threads for compiling in parallel (depending on how many cores/how much memory your RPi has)  
make -j2  
```  
  
This will take a long time.  
  
    
  
If you want to install emulationstation to  `/usr/local/bin/emulationstation`, which will let you just type 'emulationstation' to run it, you can do:  
  
```  
sudo make install  
```  
  
**NOTE:**  This will conflict with RetroPie, which installs a bash script to  `/usr/bin/emulationstation`.  
  
    
  
Otherwise, you can run the binary from the root of the EmulationStation folder:  
  
```  
../emulationstation  
```  
  
    
  
**Reset GPU RAM to normal values and reboot**  
  
```  
sudo nano /boot/config.txt  
# change/add "gpu_mem = 32" to "gpu_mem = 128" or "gpu_mem = 256", depending on your Pi model  
sudo reboot  
```  
  
    
  
[Configure EmulationStation](https://emulationstation.org/gettingstarted.html#config)  and install some themes.  
  
----------