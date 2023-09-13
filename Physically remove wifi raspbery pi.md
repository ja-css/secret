# Disabling WiFI/Bluetooth by hardware [ From Raspberry Pi Zero W to 3B+ ]  
#### See pi4 below  
  
https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware  
  
Index:  
  
1.  Intro.  
2.  CYW/BCM43438 schematics.  
3.  X-ray images and WLBGA ball map (colored).  
4.  Images showing what to remove and extra.  
5.  Raspberry Pi Zero 2 W shield information.  
6.  Pliers to remove the SMD inductor and video.  
7.  Check by terminal if WiFi and Bluetooth are disconnected.  
8.  [NEW] Scheme with examples to disable WiFi and Bluetooth from different layers.  
  
----------  
  
## Intro:  
  
One of the biggest problems are that Raspberry Pi do not show the schematics of the wireless. See this example:  [https://datasheets.raspberrypi.com/rpizero/raspberry-pi-zero-w-reduced-schematics.pdf](https://datasheets.raspberrypi.com/rpizero/raspberry-pi-zero-w-reduced-schematics.pdf)  
  
Thanks that we got access to a X-ray picture of the wireless chip, I tried to figure out some of the circuits. Not easy because that PCB is multilayer. So I just figured out some of them.  
  
The best news are that RPi Zero w and RPi Zero 2 W wireless chip still in the same position, is not rotated and in both boards we should remove exactly the same SMD component. Cool!, isn't it? 🙂  
  
I didn't end up removing  [VDD](https://en.wikipedia.org/wiki/IC_power-supply_pin)  becasuse I can't see well the all layers and to me seems like  [VDD](https://en.wikipedia.org/wiki/IC_power-supply_pin)  comes from another PCB layer under the chip.  
  
----------  
  
## CYW/BCM43438 schematics:  
  
#### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#basic-schematics-showing-were-we-are-going-to-get-rid-of-the-smd-inductor)Basic schematics showing were we are going to get rid of the SMD inductor.  
  
[![image.png](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/raw/main/images/1.png)](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/blob/main/images/1.png)  
  
Documentation source: ([https://www.cypress.com/file/298076/download](https://www.cypress.com/file/298076/download))  
  
----------  
  
## X-ray images and WLBGA ball map (colored):  
  
### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#-orange-big-dot-made-as-position-reference)🟠 Orange big dot made as position reference.  
  
#### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#here-we-have-the-x-ray-image-and-normal-one-to-let-you-know-the-actual-position)Here we have the X-ray image and normal one to let you know the actual position.  
  
[![](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/raw/main/images/2.png)](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/blob/main/images/2.png)  
  
X-ray image by  **@Vilas1979**  
  
[![](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/raw/main/images/3.png)](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/blob/main/images/3.png)  
  
----------  
  
## Images showing what to remove and extra:  
  
### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#image-showing-what-to-remove-to-disable-wifibluetooth-by-removing-the-inductor-that-goes-to-the-pin-sr_vlx)Image showing what to remove to disable WiFi/Bluetooth by removing the inductor that goes to the pin  **SR_VLX**:  
  
Information about  **SR_VLX**:  [![](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/raw/main/images/4.png)](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/blob/main/images/4.png)  
  
VDD and GND still connected into the board but chip is not able to work without the inductor. But don't worry:  
  
💡  **As a simile**: Having a car with 1 of 3  [ABS/brakes](https://en.wikipedia.org/wiki/Anti-lock_braking_system)  cable broken. The engine will start, but the ECU will detect that the  [ABS](https://en.wikipedia.org/wiki/Anti-lock_braking_system)  is not reachable. So is impossible to get connected to it, transmit any data and use that component.  
  
[![](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/raw/main/images/5.png)](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/blob/main/images/5.png)  
  
### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#extra-if-you-got-the-case-that-you-would-like-just-to-remove-the-wifi-just-remove-this-component)EXTRA: If you got the case that you would like just to remove the WiFi, just remove this component:  
  
[![](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/raw/main/images/6.png)](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/blob/main/images/6.png)  
  
### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#raspberry-pi-zero-2-w-metal-shield-information)Raspberry Pi Zero 2 W metal shield information:  
  
This is the metal shield that contains the wireless chip. You should remove it, in order to access to the SMD components:  
  
#### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#you-can-use)You can use:  
  
To remove the whole shield:  
  
1.  Heat gun  
2.  Hair dryer  
  
To break a part of the shield:  
  
1.  Dremel  
2.  Sand paper  
  
[![](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/raw/main/images/metalshield.jpg)](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/blob/main/images/metalshield.jpg)  
  
Once you have removed it you will see this (marked in red, the component to be removed):  
  
[![](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/raw/main/images/rpiz2w.jpg)](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/blob/main/images/rpiz2w.jpg)  
  
## Remove SMD inductor  
  
### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#easy-with-pliers)Easy: with pliers:  
  
[![drawing](https://user-images.githubusercontent.com/52879067/166446926-0cdae21c-49ea-4af2-9c9f-4160f7cba173.png)](https://user-images.githubusercontent.com/52879067/166446926-0cdae21c-49ea-4af2-9c9f-4160f7cba173.png)  
  
### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#video-showing-how-to-remove-it-with-pliers)Video showing how to remove it with pliers:  
  
[![Qries](https://user-images.githubusercontent.com/52879067/166448285-b4b02f5a-7335-4521-8950-f41f29067419.png)](https://estudiobitcoin.com/wp-content/uploads/2022/03/Is-working-and-is-very-easy-and-simple-with-pliers.-%EF%B8%8F-Just-go-and-cut-in-the-middle.-Here-goes-the-video-made-by-@j4vl.-Thanks-@Seed.mp4)  
  
### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#-video-of-the-process)📺 Video of the process:  
  
-   [Video by @SeedSigner](https://video.twimg.com/amplify_video/1616914307140165635/vid/750x640/sTZKr1mknSnGfy0g.mp4?tag=16)  
  
### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#text-tutorial-just-using-desoldering-iron)Text tutorial just using desoldering iron:  
  
-   Get soldering iron quite warm. Put the soldering iron on top of each extremes (first one, and then the other) of the SMD and push a bit to one side.  
  
----------  
  
## Check by terminal if WiFi and Bluetooth are disconnected.  
  
#### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#wifi)WiFi  
  
Once you boot the system, open Terminal and type:  
  
-   `ip addr`  
  
If there is not  **wlan0**, WiFi is disabled by hardware.  
  
#### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#bluetooth)Bluetooth  
  
Open Terminal and type:  
  
-   `bluetoothctl`  
  
If bluetoothctl do not open or seems to be frozen, is good signal... But if open type:  
  
-   `scan on`  
  
If you can see values (Probably not), write me back.  
  
Enjoy!  
  
----------  
  
## NEW scheme with examples to disable WiFi and Bluetooth from different layers.  
  
[![](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/raw/main/images/schema_en.png)](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware/blob/main/images/schema_en.png)  
  
As a note: If a new kernel is compiled without the WiFi or Bluetooth modules, as well as disabling certain components, it is not necessary to use any of the commands shown above.  
  
----------  
  
If you like it you can support me:  [On-Chain ⛓️ or ⚡️ Lightning network](http://btcpay.desobedientetecnologico.com/)  
  
###### [](https://github.com/DesobedienteTecnologico/rpi_disable_wifi_and_bt_by_hardware#license-cc-by-40)License:  **CC BY 4.0**  
  
  
# Pi 4  
  
[](https://raspberrypi.stackexchange.com/posts/114605/timeline)  
  
If you want to physically disable Wifi/BT, the easiest way is to disconnect the antenna, or short the antenna output to ground via a 1nF capacitor.  
  
[![enter image description here](https://i.stack.imgur.com/O4m2q.jpg)](https://i.stack.imgur.com/O4m2q.jpg)  
  
The capacitor is there to avoid having an actual short to ground, which could blow the chip up. 1nF is only 0.06 Ohm at 2.5GHz, so for RF purposes it's as good as a short. A typical antenna impedance is 50 Ohm. Having a 0.06 Ohm path to ground means that only about 0.1% of the transmitter signal will reach the antenna, while 99.9% will be lost. That's about 60 dB attenuation.  
  
Considering the WiFi signal already starts at about -30dBm, attenuation by another 60 will bring the signal down to -90 dBm, which is barely above the  [thermal noise](https://en.wikipedia.org/wiki/Johnson%E2%80%93Nyquist_noise). This means a few meters away from the device (another -10 dB), you won't be able to receive the signal even theoretically.  
  
Removing the WiFi/BT module is trivial with the right equipment (heat up the board and lift the shield and the IC up with a vacuum pen), but without a schematic I would not give you any guarantees that the rest of the Pi will work once the WiFi chip is gone. And even if the schematic is released, it would be quite a task to analyse it and create a modified device tree for the kernel to correctly understand the new hardware configuration.