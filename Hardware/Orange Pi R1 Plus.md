# Orange Pi R1 Plus  
  
#### More info http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_R1_Plus  
#### More info https://learn.adafruit.com/circuitpython-on-orangepi-linux -> https://learn.adafruit.com/circuitpython-on-orangepi-linux/orange-pi-r1  
  
  
## Orange Pi R1 Setup  
  
This page is for the Orange Pi R1 only!  
  
Once armbian is installed, its easy to tell what board you have, simply  **cat /etc/armbian-release**  and look for  `BOARD_NAME`  
  
[![sensors_cat-armbian-release.png](https://cdn-learn.adafruit.com/assets/assets/000/076/162/medium800/sensors_cat-armbian-release.png?1559106134)](https://learn.adafruit.com/assets/76162)  
  
---   
  
Install ARMbian on your Orange Pi  
  
We're only going to be using armbian, other distros could be made to work but you'd probably need to figure out how to detect the platform since we rely on  `/etc/armbian-release`  existing.  
  
Download and install the latest armbian, for example we're using  [https://www.armbian.com/orange-pi-r1/](https://www.armbian.com/orange-pi-r1/)  
  
There's some documentation to get started at  [https://docs.armbian.com/User-Guide_Getting-Started/](https://docs.armbian.com/User-Guide_Getting-Started/)  
  
Blinka only supports ARMbian because that's the most stable OS we could find and it's easy to detect which board you have  
  
[![sensors_orangepir1.png](https://cdn-learn.adafruit.com/assets/assets/000/076/163/medium640/sensors_orangepir1.png?1559106439)](https://learn.adafruit.com/assets/76163)  
  
We've found the easiest way to connect is through a console cable, wired to the debug port, and then on your computer, use a serial monitor at 115200 baud. For the Orange Pi R1, you will also need to solder a header for debugging and GPIO access to the Orange Pi R1.  
  
![Color Coded 2x20 Header for Raspberry Pi](https://cdn-shop.adafruit.com/640x480/3907-00.jpg)  
  
[Color Coded Header for Raspberry Pi](https://www.adafruit.com/product/3907)  
  
Here's a snazzy accessory for your Raspberry Pi Zero or Zero W, a color-coded 2x20 header! This Color Coded Header takes the mystery out of your Raspberry...  
  
$1.75  
  
In Stock  
  
[Add to Cart](https://www.adafruit.com/product/3907)  
  
#   
  
Connecting the GPIO/Debug Header  
  
Because the Orange Pi R1 does not come with any GPIO headers, you will need to solder one in order to connect breakout boards.  
  
[![sensors_assembly_1.png](https://cdn-learn.adafruit.com/assets/assets/000/076/170/medium640/sensors_assembly_1.png?1559111923)](https://learn.adafruit.com/assets/76170)  
[![sensors_assembly_1.png](https://cdn-learn.adafruit.com/assets/assets/000/076/170/thumb160/sensors_assembly_1.png?1559111923)](https://learn.adafruit.com/assets/76170)  
[![sensors_assembly_2.png](https://cdn-learn.adafruit.com/assets/assets/000/076/171/thumb160/sensors_assembly_2.png?1559111963)](https://learn.adafruit.com/assets/76171)  
  
## Prepare the header strip:  
  
Cut the strip to length. You will want to make a cut to the side of the blue strip that is closer to the center. After cutting, you should have one piece that is 2x13 pins and another that is 2x7 pins.  
  
[![sensors_assembly_2b.png](https://cdn-learn.adafruit.com/assets/assets/000/076/174/medium640/sensors_assembly_2b.png?1559112471)](https://learn.adafruit.com/assets/76174)  
  
## Place the header:  
  
Because the ethernet ports are on the tall side, we found that placing a small object under the header such as a small piece of foam works well.  
  
[![sensors_assembly_3.png](https://cdn-learn.adafruit.com/assets/assets/000/076/175/medium640/sensors_assembly_3.png?1559112493)](https://learn.adafruit.com/assets/76175)  
  
## And Solder!  
  
Be sure to solder all pins for reliable electrical contact.    
    
_(For tips on soldering, be sure to check out our_ [_Guide to Excellent Soldering_](http://learn.adafruit.com/adafruit-guide-excellent-soldering)_)._  
  
[![sensors_assembly_4.png](https://cdn-learn.adafruit.com/assets/assets/000/076/176/medium640/sensors_assembly_4.png?1559112534)](https://learn.adafruit.com/assets/76176)  
  
Check your solder joints visually and continue onto the next steps  
  
[![sensors_assembly_5.png](https://cdn-learn.adafruit.com/assets/assets/000/076/177/medium640/sensors_assembly_5.png?1559112573)](https://learn.adafruit.com/assets/76177)  
  
## You're done!  
  
Continue on to the next steps.  
  
# Logging in using Serial  
  
To log in through serial, you will need to connect a USB to TTL Serial cable up to the Orange Pi. You may also need to install some software. Be sure to check out our [Adafruit's Raspberry Pi Lesson 5. Using a Console Cable](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-5-using-a-console-cable) guide. Although it is written for a Raspberry Pi, the drivers and cable connection should be the same.  
  
## Software Installation (Linux)  
  
Linux Kernels 2.4.31 and above already have the PL2303 and CP210X USB driver for the Console Lead built-in, so you should not need to install that.    
    
Some distributions such as Ubuntu 12.10 do not include the "screen" command. Try running the command "screen" and if you get an error message, you can install it by typing the following command:  
  
`sudo apt-get install screen`  
  
## Cable connection  
  
![USB to TTL Serial Cable With Type A plug and 4 wire sockets](https://cdn-shop.adafruit.com/640x480/954-02.jpg)  
  
[USB to TTL Serial Cable - Debug / Console Cable for Raspberry Pi](https://www.adafruit.com/product/954)  
  
[![sensors_serial_cable.png](https://cdn-learn.adafruit.com/assets/assets/000/076/380/medium640/sensors_serial_cable.png?1559603196)](https://learn.adafruit.com/assets/76380)  
  
Connect the serial cable's TX (White), RX (Green), and Ground (Black) wires to the 3 pins next to the ethernet ports as shown in the photo and start your terminal program.  
  
## Linux terminal program  
  
If you are using Linux, its much like the above but often times the device is called **/dev/ttyUSB0**  - you may want to run **sudo dmesg**  after plugging in and looking for hints on what the device is called.  
  
Then use the command:  
`sudo screen /dev/ttyUSB0 115200`  
  
To start communication with the Pi, press ENTER and you should see the login prompt from the Pi.  
  
## Interacting via terminal  
  
[![sensors_serial_cable_linux.png](https://cdn-learn.adafruit.com/assets/assets/000/076/381/medium640/sensors_serial_cable_linux.png?1559605584)](https://learn.adafruit.com/assets/76381)  
  
Once powered correctly and with the right SD card you should get a login prompt  
  
[![sensors_new_user.png](https://cdn-learn.adafruit.com/assets/assets/000/076/385/medium640/sensors_new_user.png?1559606643)](https://learn.adafruit.com/assets/76385)  
  
After logging in you may be asked to create a new username, we recommend  **pi**  - if our instructions end up adding gpio access for the pi user, you can copy and paste em  
  
[![sensors_PostLoginScreen.png](https://cdn-learn.adafruit.com/assets/assets/000/076/386/medium800/sensors_PostLoginScreen.png?1559607082)](https://learn.adafruit.com/assets/76386)  
  
Once installed, you may want to enable mDNS so you can  `ssh  pi@orangepi.local`  instead of needing to know the IP address:  
  
-   `sudo apt-get install avahi-daemon`  
  
then  **reboot**  
  
#   
  
Connecting using DHCP and SSH  
  
Another option that you may be available to you is to connect using SSH, or Secure Shell. To connect, you will need to have DHCP, or Dynamic Host Configuration Protocol, enabled and configured on your network and look up the IP address assigned to the Orange Pi R1.  
  
By default, most routers have DHCP enabled and some routers will display all of the hosts connected to them through the web configuration panel. If you are able to retrieve the IP address, then you can connect through SSH.  
  
##   
  
MacOS/Linux  
  
Mac and Linux should have SSH installed by default and you can access it by going to the command line and typing  `ssh  root@192.168.0.25`if the IP address of your Orange Pi is 192.168.0.25.  
  
##   
  
Windows  
  
The easiest way to connect with windows is to use a Terminal program called PuTTY from [https://www.putty.org](https://www.putty.org/).  
  
You can read more about connecting through SSH in our [Adafruit's Raspberry Pi Lesson 6. Using SSH](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-6-using-ssh) guide. Although this is written for Raspberry Pi, the steps should be the same.  
  
#   
  
Set your Python install to Python 3 Default  
  
There's a few ways to do this, we recommend something like this  
  
[Copy Text](https://learn.adafruit.com/circuitpython-on-orangepi-linux/orange-pi-r1#)  
  
sudo apt-get install -y python3 git python3-pip  
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1  
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.5 2  
sudo update-alternatives --config python  
  
Of course change the version numbers if newer Python is distributed  
  
#   
  
Install libgpiod  
  
libgpiod is what we use for gpio toggling. To install run this command:  
  
[Copy Text](https://learn.adafruit.com/circuitpython-on-orangepi-linux/orange-pi-r1#)  
  
sudo apt-get install libgpiod2 python3-libgpiod  
pip3 install gpiod  
  
After installation you should be able to `import gpiod` from within Python3  
  
[![sensors_libgpiod_import.png](https://cdn-learn.adafruit.com/assets/assets/000/076/409/medium800/sensors_libgpiod_import.png?1559609251)](https://learn.adafruit.com/assets/76409)  
  
#   
  
Install Required pip3 Modules  
  
For the Orange Pi R1, a couple of pip modules are required to smooth out the installation.  
  
Run the following command.  
  
`sudo pip3 install wheel flask`  
  
#   
  
Update Your Board and Python  
  
Run the standard updates:  
  
`sudo apt-get update`  
  
`sudo apt-get upgrade`  
  
and  
  
`sudo pip3 install --upgrade setuptools`  
  
Update all your python 3 packages with  
  
`pip3 freeze - local | grep -v '^\-e' | cut -d = -f 1 | xargs -n1 pip3 install -U`  
  
and  
  
`sudo bash`  
  
`pip3 freeze - local | grep -v '^\-e' | cut -d = -f 1 | xargs -n1 pip3 install -U`  
  
#   
  
Enable UART, I2C and SPI  
  
A vast number of our CircuitPython drivers use UART, I2C and SPI for interfacing so you'll want to get those enabled.  
  
You only have to do this _once_ per board but by default all three interfaces are disabled!  
  
[![sensors_i2cspisetup.png](https://cdn-learn.adafruit.com/assets/assets/000/076/470/medium640/sensors_i2cspisetup.png?1559663648)](https://learn.adafruit.com/assets/76470)  
  
Install the support software with  
  
-   `sudo apt-get install -y python-smbus python-dev i2c-tools`  
-   `sudo adduser pi i2c`  
  
[![sensors_armbian-release.png](https://cdn-learn.adafruit.com/assets/assets/000/076/471/medium640/sensors_armbian-release.png?1559663849)](https://learn.adafruit.com/assets/76471)  
  
Read **/etc/armbian-release** to figure out your board family, in this case its a **sun8i**  
  
Edit **/boot/armbianEnv.txt** and add these lines at the end, adjusting the `overlay_prefix` for your particular board  
  
`overlay_prefix=sun8i-h3    
overlays=uart1 i2c0 spi-spidev usbhost2 usbhost3    
param_spidev_spi_bus=1`  
  
[![sensors_armbianEnv.png](https://cdn-learn.adafruit.com/assets/assets/000/076/477/medium800/sensors_armbianEnv.png?1559674398)](https://learn.adafruit.com/assets/76477)  
  
Once you're done with both and have rebooted, verify you have the I2C and SPI devices with the command `ls /dev/i2c* /dev/spi*`  
  
You should see at least one i2c device and one spi device  
  
[![sensors_i2c-device-list.png](https://cdn-learn.adafruit.com/assets/assets/000/076/479/medium800/sensors_i2c-device-list.png?1559675376)](https://learn.adafruit.com/assets/76479)  
  
[![sensors_i2c-device-details.png](https://cdn-learn.adafruit.com/assets/assets/000/076/480/medium800/sensors_i2c-device-details.png?1559675485)](https://learn.adafruit.com/assets/76480)  
  
[![sensors_i2cdetect.png](https://cdn-learn.adafruit.com/assets/assets/000/076/481/medium640/sensors_i2cdetect.png?1559675696)](https://learn.adafruit.com/assets/76481)  
  
You can test to see what I2C addresses are connected by running `sudo i2cdetect -y 0`(on **PA11/PA12**)  
  
In this case I do have a sensor on the 'standard' i2c port i2c-0 under address 0x77  
  
You can also install WiringOP-Zero and then run `gpio readall` to see the MUX status. If you see **ALT3** next to a pin, it's a plain GPIO. If you see **ALT4** or **ALT5**, its an SPI/I2C/UART peripheral  
  
[![sensors_gpio_readall.png](https://cdn-learn.adafruit.com/assets/assets/000/076/546/medium800/sensors_gpio_readall.png?1559925030)](https://learn.adafruit.com/assets/76546)  
  
#   
  
Install Python libraries  
  
Now you're ready to install all the python support  
  
Run the following command to install **adafruit_blinka**  
  
`sudo pip3 install adafruit-blinka`  
  
[![sensors_install_blinka.png](https://cdn-learn.adafruit.com/assets/assets/000/076/485/medium800/sensors_install_blinka.png?1559681350)](https://learn.adafruit.com/assets/76485)  
  
The computer will install a few different libraries such as `adafruit-pureio` (our ioctl-only i2c library), `spidev` (for SPI interfacing), `Adafruit-GPIO` (for detecting your board) and of course `adafruit-blinka`  
  
That's pretty much it! You're now ready to test.  
  
Create a new file called **blinkatest.py** with **nano** or your favorite text editor and put the following in:  
  
[Download File](https://learn.adafruit.com/pages/16236/elements/3029935/download)  
  
[Copy Code](https://learn.adafruit.com/circuitpython-on-orangepi-linux/orange-pi-r1#)  
  
import board  
import digitalio  
import busio  
  
print("Hello blinka!")  
  
# Try to great a Digital input  
pin = digitalio.DigitalInOut(board.PA6)  
print("Digital IO ok!")  
  
# Try to create an I2C device  
i2c = busio.I2C(board.SCL, board.SDA)  
print("I2C ok!")  
  
# Try to create an SPI device  
spi = busio.SPI(board.SCLK, board.MOSI, board.MISO)  
print("SPI ok!")  
  
print("done!")  
  
Save it and run at the command line with  
  
`sudo python3 blinkatest.py`  
  
You should see the following, indicating digital i/o, I2C and SPI all worked  
  
[![sensors_test.png](https://cdn-learn.adafruit.com/assets/assets/000/076/487/medium800/sensors_test.png?1559681480)](https://learn.adafruit.com/assets/76487)

