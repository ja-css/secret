# How to install Linux on the eMMC of an Orange Pi 3 LTS
(from https://jumptuck.com/blog/2023-02-13-install-linux-orange-pi-3-lts-emmc/#:~:text=Download%20Ubuntu%20and%20Flash%20onto%20an%20SD%20Card,chose%20your%20SD%20card%2C%20and%20click%20on%20flash.)


[1. Intro](#intro)
[2. Download Ubuntu and Flash onto an SD Card for installation](#download-ubuntu-and-flash-onto-an-sd-card-for-installation)
[3. Boot in Linux and Install on the eMMC Flash Memory](#boot-in-linux-and-install-on-the-emmc-flash-memory)
[4. Bonus: Connecting to WiFi and fixing the DNS](#bonus-connecting-to-wifi-and-fixing-the-dns)
[5. Disable WiFi](#disable-wifi)
[6. Wrapping Up](#wrapping-up)

## 1. Intro

It's impossible to buy a Raspberry Pi right now so my company is looking to the Orange Pi as an alternative. One perk is that these boards include eMMC memory so that the OS is stored on the board itself. Here's how to install Linux to the flash memory.

![How to install Linux on the eMMC of an Orange Pi 3 LTS](https://jumptuck.com/wp-content/uploads/2023/02/orange_pi_3_lts.jpg)

At Golioth,  [we use Hardware in the Loop (HIL)](https://blog.golioth.io/golioth-hil-testing-part1/)  testing to validate all pull requests to the platform on the actual hardware being targeted. This means that GitHub actions are compiling and running code on the nRF9160, ESP32, and a few other platforms. It’s really slick!

Part of making this all work is a Linux box that connects GitHub actions to the actual dev boards. It’s a perfect job for a Raspberry Pi. The catch, of course, is that because of the chip shortage we haven’t been able to buy a Raspberry Pi for the last year and a half.

This weekend I spent some time validating  [the Orange Pi 3 LTS](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/orange-pi-3-LTS.html)  as a suitable alternative, and I’m quite happy with it. One of the only drawbacks I have found is that the documentation is not nearly as complete. It took me a while to find the correct command for copying Linux to the onboard eMMC flash memory:

```
$ nand-sata-install
```

This is command is nowhere to be found on the official wiki, and the documentation available for download includes a deprecated command that no longer works. There is also a bit more that goes into the install so I’m going to walk through the process in this tutorial.

## 2. Download Ubuntu and Flash onto an SD Card for installation

The first step is to download an Ubuntu image and flash it onto an SD Card. We will boot the Orange Pi 3 LTS from this SD Card and then install the OS onto the eMMC storage.

1.  Go to  [the Orange Pi downloads](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-pi-3-LTS.html)  page and choose Ubuntu.
    
    ![Download Orange Pi Ubuntu image](https://jumptuck.com/wp-content/uploads/2023/02/orangepi-image-download.png)
    
    You will be redirected to a Google Drive folder with several files in it. I downloaded the  `Orangepi3-lts_3.0.8_ubuntu_jammy_server_linux5.16.17.7z`  archive as I don’t need a desktop for this project. However, installation for any of these images is largely the same.
    
2.  Extract the  `.img`  file from the archive you just downloaded
3.  Download  [Balena Etcher](https://jumptuck.com/blog/2023-02-13-install-linux-orange-pi-3-lts-emmc/aria-label=%22Compressed%20Archive:Orangepi3-lts_3.0.8_ubuntu_jammy_server_linux5.16.17.7z%22data-tooltip-align=%22b,c%22%20data-tooltip-delay=%22500%22data-tooltip-unhoverable=%22true%22%3EOrangepi3-lts_3.0.8_ubuntu_jammy_server_linux5.16.17.7z%3C/div%3E%3C/div%3E%60), which is useful for flashing installation images to an SD Card
    
    ![Balena Etcher](https://jumptuck.com/wp-content/uploads/2023/02/orangepi-balena-etcher.png)
    
4.  Select your  `.img`  file, chose your SD card, and click on flash.

I have seen mention that this SD card should be smaller than the 8GB eMMC on which we will be installing the OS. However, I haven’t verified this requirement. I happened to be using a 2GB card and it works well.

## 3. Boot in Linux and Install on the eMMC Flash Memory

Place your SD card in the slot on the bottom of the Oragne Pi 3 LTS board. Connect your monitor via HDMI and your keyboard using the USB ports. Apply power to the USB-C connector on the board.

1.  After boot, log in with username  `root`  and password  `orangepi`  ![Orange Pi boot screen](https://jumptuck.com/wp-content/uploads/2023/02/orangepi-login.png)
    
2.  Type  `nand-sata-install`  to begin the installation process

4.  Choose to boot from eMMC, choose  `EXT4`, then accept the warning that the process will erase all contents of the eMMC memory  ![eMMC install menu](https://jumptuck.com/wp-content/uploads/2023/02/orangepi-emmc-install.png)
    
5.  After a few minutes you will be given the option to power down the board. Do so and remove the SD Card.

Installation of Linux on the eMMC is now complete

## 4. Bonus: Connecting to WiFi and fixing the DNS

At this point Linux is installed and working on the Orange Pi. You can boot the board and log in with  `root`/`orangepi`. If you are using an Ethernet connection you likely already have internet working, but I only wanted to use WiFi. Let’s step through the process of setting that up.

1.  Add WiFi to the network interfaces
    
    ```
     $ sudo nano /etc/network/interfaces
    
     #Add the following to the bottom of this file:
     auto wlan0
     iface wlan0 inet dhcp
     wpa-ssid your-wifi-ssid
     wpa-psk your-wifi-password
    
    ```
    
2.  WiFi will work after a reboot but my DNS wasn’t working. To fix this, edit the systemd resolved configuration:
    
    ```
     $ sudo nano /etc/resolv.conf
    
     #Add the folling to the bottom of this file:
     search domain.name
     nameserver 8.8.8.8
     nameserver 1.1.1.1
     nameserver 1.0.0.1
    
    ```
    
3.  Reboot the board by typing  `reboot`.

## 5. Disable WiFi

(from https://forum.armbian.com/topic/12127-orange-pi-zero-512mb-disable-wifi-and-usb/#:~:text=Disable%20in%20armbian-config%20-%3E%20hardware%20and%20blacklist,wireless%20module%20%28name%20is%20xradio_wlan%20or%20something%29)

Disable in armbian-config -> hardware and [blacklist](https://askubuntu.com/questions/110341/how-to-blacklist-kernel-modules) wireless module (name is xradio_wlan or something)

for blacklisting
- xradio_wlan
- mac80211
- cfg80211

---

(from https://www.reddit.com/r/OrangePI/comments/tvvxfw/how_can_i_disable_the_wifi_function_of_orange_pi/)

You can blacklist kernel wifi modules so it'll be totally disabled:

In directory /etc/modprobe.d/ create 3 files for each of modules:
`echo 'blacklist xradio_wlan' > /etc/modprobe.d/xradio_wlan.conf`
`echo 'blacklist mac80211' > /etc/modprobe.d/mac80211.conf`
`echo 'blacklist cfg80211' > /etc/modprobe.d/cfg80211.conf`

Check that files were created:
`cat /etc/modprobe.d/*.conf`

Output should be like that:
`blacklist xradio_wlan`
`blacklist mac80211`
`blacklist cfg80211`
Reboot, and after reboot you can check that wireless interface disappeared from  `ip a`  👍



## 6. Wrapping Up

The Orange Pi is now running from the onboard eMMC, it’s connected to your WiFi network, and it’s able to use a DNS server for domain name lookup. There is still some work to do. I recommend you add a public key for logging in over SSH and disable password-based authentication. But that’s a post for another day.
