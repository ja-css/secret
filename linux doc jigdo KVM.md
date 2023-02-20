
**Preparation**
[1. Debian with firmware (image)](#debian-with-firmware-image)
[2. Debian DVDs and jigdo](#debian-dvds-and-jigdo)
[3. Checksums](#checksums)
**Installation**
[4. Adding new DVD repos to debian](#adding-new-dvd-repos-to-debian)
**Host OS configuration**
[5. Disable autoplay for removable media (Host and Guest)](#disable-autoplay-for-removable-media-host-and-guest)
[6. Add user to sudoers](#add-user-to-sudoers)
[7. Install OpenVPN](#install-openvpn)
[8. Iptables config](#iptables-config)
[9. Install KVM](#install-kvm)
[10. Splitting files into chunks and merging back](#splitting-files-into-chunks-and-merging-back)
[11. Attach usb (Yubikey) to qemu](#attach-usb-yubikey-to-qemu)
[12. Useful debug command line utils: speedtest, show ip](#useful-debug-command-line-utils-speedtest-show-ip)
**Strengthening your defenses**
[13. Guest: set duckduckgo as default search engine](#guest-set-duckduckgo-as-default-search-engine)
[14. Double-conceal your connections with nested OVPN tunnel](#double-conceal-your-connections-with-nested-ovpn-tunnel)
[15. Hardware tokens, keys and managers](#hardware-tokens-keys-and-managers)
[16. Auto-reConnector/Randomizer](#auto-reconnectorrandomizer)

# Preparation

## 1. Debian with firmware (image)

(info from here: https://cdimage.debian.org/debian-cd/current/amd64/jigdo-dvd/)

An image of Debian with firmware embedded is available:

    Non-free Firmware
    This is an  **official**  Debian image build and so only includes Free Software.

-> (**FIRMWARE DVD IS HERE**) https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware
*) ISO: https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/11.6.0+nonfree/amd64/iso-dvd/
*) TorrentC: https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/11.6.0+nonfree/amd64/bt-dvd/
`apt install transmission transmission-cli`

Theoretically you can try to find your firmware and create a flash drive with it, so that you can use an "official" installation disk, there are places like:
https://github.com/wkennington/linux-firmware
git clone git://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git

Practically, however, I tried and failed to do it from my first couple of tries.
So if you have no intention to learn this aspect in depth, just use the firmware disk.

## 2. Debian DVDs and jigdo

Only the first disk can be downloaded directly, additional disks are available via jigdo:
#### 1. install jigdo:
`brew install jigdo`
or
`sudo apt-get install jigdo-file`

#### 2. find jigdo disk list:
from here: https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/

    Only the first few images are available! Where are the rest?

-> https://www.debian.org/CD/faq/#why-jigdo (look at the top right of the whole page)
```
    Download
       Download with Jigdo
```
-> https://www.debian.org/CD/jigdo-cd/#which
```
    Official images
        Official jigdo files for the  stable  release
```
-> (**THE LIST IS HERE**) https://cdimage.debian.org/debian-cd/current/amd64/jigdo-dvd/ 

#### 3. download using jigdo
Run: `jigdo-lite debian-11.3.0-amd64-DVD-3.jigdo`
Input the following parameters:
`Files to scan:` - press Enter
`...`
`Debian mirror [http://ftp.us.debian.org/debian/]:` http://ftp.us.debian.org/debian/

more info here: https://www.debian.org/CD/jigdo-cd/#how

#### 4. firmware disk

## 3. Checksums
`openssl sha256 FILENAME`
`openssl sha512 FILENAME`


# Installation

## 4. Adding new DVD repos to debian
Practically, I had problems adding it post-install, much preferably to add it while installing, when asked.
Try not to register Official disk 1 and Firmware disk 1 together, only register the one that's used for installation.
To avoid cd tray issue, first press enter, and quickly after that "open tray" button on CD-ROM, and only in this order.

But theoretically, it can be done in this way after installation:

`# apt-cdrom add`

OR UI : In Debian (gnome):
Software 
-> 
/// (button in the right top corner, 3 horizontal lines) 
-> 
Software Repositories 
->
Other Software
->
Add Volume (adds disk from CDROM)



# Host OS configuration

## 5. Disable autoplay for removable media (Host and Guest)
Debian and Ubuntu: Settings -> Removable Media, select "Do Nothing"
Check the checkbox: "Never prompt or start programs on media insertion".

## 6. Add user to sudoers
Apparently, the following 2 things need to be done:
1) Added user john to sudo group:
`sudo usermod -aG sudo john`

2) Added the following line to /etc/sudoers
`john    ALL=(ALL:ALL) ALL`

## 7. Install OpenVPN
https://wiki.debian.org/OpenVPN#Installation
`sudo apt install openvpn`

## 8. Iptables config
The goal is to have all traffic stop when OpenVPN is not connected.

1) Drop all packets
```
$ sudo iptables -P INPUT DROP
$ sudo iptables -P OUTPUT DROP
$ sudo iptables -P FORWARD DROP

$ sudo ip6tables -P INPUT DROP
$ sudo ip6tables -P OUTPUT DROP
$ sudo ip6tables -P FORWARD DROP
```

2) General INPUT rule to allow ANY and ALL locally originating (as in, on the machine running iptables) traffic.
DNS, HTTP, etc... all of it. Any connection initiated by the server running iptables should be allowed.
```
$ sudo iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

3) Allow OpenVPN
```
$ sudo iptables -A OUTPUT -p udp --dport {open_vpn_port} -d {open_vpn_ip_address} -j ACCEPT  
```

4) Enable persistence for iptables
EITHER install - `sudo apt install iptables-persistent`
OR create file `/etc/rc.local`, with the following code
```
#!/bin/bash
sudo iptables-restore /etc/iptables/rules.v4
sudo ip6 tables-restore /etc/iptables/rules.v6
```
And make it executable:
`sudo chmod a+x /etc/rc.local `
Then, you can make your current settings persistent by running:
```
$ sudo iptables-save > /etc/iptables/rules.v4
$ sudo ip6tables-save > /etc/iptables/rules.v6
```

5) Change DNS settings
First, change it in UI - connection settings, restart connection.
Double-check `/etc/resolv.conf`. If the DNS settings didn't change there, try the following:
- `/etc/resolv.conf`  is actually a symlink (`ls -l /etc/resolv.conf`) which points to  `/run/systemd/resolve/stub-resolv.conf`  (127.0.0.53) by default in Ubuntu 18.04.
- Change the symlink to point to  `/run/systemd/resolve/resolv.conf`, which lists the real DNS servers:  
    `sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf`

Google Public DNS
- 8.8.8.8, 8.8.4.4
- 2001:4860:4860::8888, 2001:4860:4860::8844

CloudFlare Public DNS
- 1.1.1.1, 1.0.0.1
- 2606:4700:4700::1111, 2606:4700:4700::1001

Yandex.DNS
- 77.88.8.8, 77.88.8.1
- 2a02:6b8::feed:0ff, 2a02:6b8:0:1::feed:0ff

6) Allow all outgoing on tun0 - OpenVPN tunnel
```
$ sudo iptables -A OUTPUT -o tun0 -j ACCEPT
```
On guest, you can restrict it to your user only:
```
$ sudo iptables -A OUTPUT -m owner --uid-owner {UID} -o tun0 -j ACCEPT
```
To determine user's *UID* by name, use the following command
`id -u {username}`

5) To see the rules on your system, you can use the following  `iptables`  command:
`$ sudo iptables -L`

6) To delete a rule:
- `sudo iptables -L --line-numbers` to show rule numbers for CHAINS
- `sudo iptables -D INPUT 3` to delete rule by CHAIN and number

7) To make iptables rules persistent after reboot:
https://linuxconfig.org/how-to-make-iptables-rules-persistent-after-reboot-on-linux
- `sudo apt install iptables-persistent` - Install  `iptables-persistent`  package
- Use  `iptables-save` and `ip6tables-save` to make rules persist after reboot
```
$ sudo iptables-save > /etc/iptables/rules.v4
$ sudo ip6tables-save > /etc/iptables/rules.v6
```
- To restore saved iptables settings, use 
```
$ sudo iptables-restore /etc/iptables/rules.v4 
$ sudo ip6tables-restore /etc/iptables/rules.v6
```

## 9. Install KVM
https://wiki.debian.org/KVM#Installation
`sudo apt install qemu-system`
`sudo apt install virt-manager`

To attach a folder to the VM in virt-manager, add a shared folder in Hardware Manager, which will ask two parameters:
`source path` - folder to share, `target path` - e.g., use "/mnt"

On Guest:
`mkdir share`
`mount -t 9p -o trans=virtio /mnt share`

**TODO:** figure out how to use snapshot mode with `virt-manager`

Use *snapshot mode* when possible, especially for entertainment VMs, so that all results of undesirable hidden malicious activity are wiped out on every restart.

Snapshot mode:
If you use the option  `-snapshot`, all disk images are considered as read only. When sectors in written, they are written in a temporary file created in  `/tmp`. You can however force the write back to the raw disk images by using the  `commit`  monitor command (or C-a s in the serial console).

https://qemu-project.gitlab.io/qemu/system/images.html#snapshot-mode

**TODO:** figure out if usb can be improved with `virt-manager`

Add the following line to the VM command:
`-usb -device usb-host,vendorid=0x1050,productid=0x0402`
Benefits :
- can unplug and replug USB device 
- will work if a device is not plugged in on VM start;
- device will still work even if plugged and replugged to different USB ports;
- multiple devices can be used at the same time;
- vendor/product values don't change, unlike hostaddr;
- also for some reason this works for Flash drive, while hostaddr doesn't.

Don't use hostaddr, which always changes
~~`-usb -device usb-host,hostbus=1,hostaddr=19`~~



## 10. Splitting files into chunks and merging back
To avoid mac error 1309 when copying large files (e.g. iso images) to flash drive
- Split in 500mb chunks
```bash
split -b 5000m MYBIGFILE
```
- On target machine merge chunks
```bash
cat x?? > MYBIGFILE
```

## 11. Attach usb (Yubikey) to qemu
Hardware manager in virt-manager doesn't allow to reconect your USB devices, so keep your USB plugged in.
Also, it won't start without the device plugged in if it's set in the config.
But otherwise it works flawlessly, flash drives are recognized by the OS and all.

## 12. Useful debug command line utils: speedtest, show ip
Avoid running any of those on Host, just in case, unless it's a test run.

On Guest - install curl, telnet

`sudo apt install speedtest-cli`
Then run `speedtest`

`telnet telnetmyip.com`
`telnet 3.19.111.8`

# Strengthening your defenses

## 13. Guest: set duckduckgo as default search engine
Update Firefox from apt `sudo apt install firefox`
In Firefox 
- update search engine; 
- no proxy or custom proxy, don't use system proxy;
- DNS over HTTPS if possible (1.1.1.1).

## 14. Double-conceal your connections with nested OVPN tunnel
You can run another OVPN client on your guest machine, effectively making your traffic go through 2 chained OVPN servers. To scale it out to chains with more than 2 hops, a different technological solution would be required, but in the context of this setup it's a quick and easy win that fits naturally in the nested hypervisor structure.

## 15. Hardware tokens, keys and managers
**TODO:**
Make sure you have as many of those as you can and use them as much as you can.

## 16. Codecs
```
sudo apt update
sudo apt install ubuntu-restricted-extras
```
optional:
 ```
sudo add-apt-repository multiverse
```


## 16. Auto-reConnector/Randomizer
**TODO:**
Random strategy after timeout on entire set, on a subset.
Constant strategy with memorization, possible pool.
Background pinging, speed tests, statistical data collection.

## 17. Init actions:
-
1. display resolution
2. disable removable media - autoplay
3. power - never sleep
4. copy vpn, walpapers
5. DNS for wired connection - also set in resolv.conf
6. set iptables settings
7. set iptables restore script
8. save iptables settings
9. Mozilla settings - search etc.
10. Remove /mnt (hardware)
-
11. set walpapers 
12.

## 18. Add a drive
- find a drive 
`sudo fdisk -l`
-  create a filesystem 
`sudo mkfs -t ext4 /dev/sdb`
- mount
`mkdir /backup`
`mount /dev/vdb /backup`
- give user permissions
`chown john /backup`