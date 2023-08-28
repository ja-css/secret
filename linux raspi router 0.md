
This is a set of instructions to set up a WiFi-OpenVPN router based on Raspberry pi.
Tested on Raspberry Pi 4 8gb, Raspberry Pi Zero W.
OS used:
```
Raspberry Pi OS Lite
-   Release date:  September 22nd 2022
-   System:  32-bit
-   Kernel version:  5.15
-   Debian version:  11 (bullseye)
```

# Disclaimer

**!!! I failed to make the following work on 64 BIT Raspbian OS!!!**
**Specifically, dhcpcd config doesn't set static wlan0 ip for some reason**

Apparently, the reason is that IPv6 is not using dhcp, and the existing router is sending some parasitic ipv6 packets (ADVERTISE?) all the time. I don't want to investigate further, because IPv4 is more than enough for my 640k.

## High level idea

```mermaid
graph LR
WiFi --> hotspot_hostapd
hotspot_hostapd --> DHCP_dnsmasq
DHCP_dnsmasq --> FORWARD_iptables
FORWARD_iptables --> openvpn_tun0
openvpn_tun0 --> Network_Of_Interest
```

## 1. Install Raspbian
To clean old data:
`sudo dd if=/dev/zero of=/dev/sda bs=512 count=1`
Unmount, unplug, replug

**Make sure you're witing an *.img file and not *.img.xz**
**Extract *.img.xz to get *.img**
`sudo dd bs=1M if=2022-04-04-raspios-bullseye-arm64-lite.img of=/dev/sda conv=fsync status=progress`

## 1.0 Set up a secure SSH
See doc `EToken, certificates, ssh` pt. 6.

## 1.1 Disable Bluetooth completely, right away (and avahi)
[Source](https://di-marco.net/blog/it/2020-04-18-tips-disabling_bluetooth_on_raspberry_pi/#disable-bluetooth-completely)

#### Uninstall  `BlueZ`  and related packages

If  `Bluetooth`  is not required at all, uninstall  `Bluetooth`  stack. It makes  `Bluetooth`  unavailable even if external  `Bluetooth`  adapter is plugged in.

This will also remove bluetooth  firmware itself.

To keep iptables
```
sudo apt install iptables
```
```
sudo apt-get purge bluez -y
sudo apt-get purge libbluetooth3
sudo apt-get autoremove -y
```
disable avahi-daemon
```
sudo systemctl disable avahi-daemon
```
check that bluetooth is gone
`dpkg -l | grep -i blue`

## 1.2 OS update and apt-get software
After installation, update OS:

Allow only outgoing traffic: `iptables-apt-update.sh`
```
# Flush
iptables -t nat -F
iptables -t mangle -F
iptables -F
iptables -X

# Block All
iptables -P OUTPUT DROP
iptables -P INPUT DROP
iptables -P FORWARD DROP
ip6tables -P OUTPUT DROP
ip6tables -P INPUT DROP
ip6tables -P FORWARD DROP

# Later SSH can be opened for local access
#iptables -A INPUT -p tcp --dport 22 -j ACCEPT


# Allow only outgoing requests for your IP for software update. Uncomment once IP is known.
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -j ACCEPT
```

Optionally switch to a mirror (consider https mirrors):
`sudo nano /etc/apt/sources.list`
listed here: https://www.raspbian.org/RaspbianMirrors
Also, to avoid the system falling back to http on archive.raspberrypi.org, set
`127.0.0.1 archive.raspberrypi.org` in /etc/hosts

Then it's safer to
`sudo apt-get update`

**UPGRADE IS OPTIONAL**
`sudo apt-get upgrade` - do we need to update/upgrade?
Upgrade comes with some "diversions" and "rpikernelhack" I REALLY don't like

And install all:
`sudo apt install hostapd dnsmasq iptables openvpn dnsutils tcpdump iftop lsof nethogs mc`

**!!! UNPLUG NETWORK AT THIS POINT!!!**

After this, make sure bluetooth wasn't reinstated.
`dpkg -l | grep -i blue`

## 2. WiFi hotspot: `hostapd`
`sudo apt-get install hostapd`


### 2.1 create config file `/etc/hostapd/hostapd.conf`
Create file
`sudo nano /etc/hostapd/hostapd.conf`

With lines:
```
interface=wlan0
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
ssid={NETWORK}
wpa_passphrase={PASSWORD}

#allow hostapd_cli
ctrl_interface=/var/run/hostapd
ctrl_interface_group=0
```
TODO: Hidden network?
TODO: Hostapd how to collect clients macs?

```
#Accept only known MAC addresses
macaddr_acl=1
accept_mac_file=/etc/hostapd/accept
```

### 2.2 register config in `/etc/default/hostapd`

`sudo nano /etc/default/hostapd`

add line
`DAEMON_CONF="/etc/hostapd/hostapd.conf"`

### 2.3 `hostapd_cli`
To get `hostapd_cli` to connect, in the hostapd.conf you need to specify
the option 'ctrl_interface=<path to hostapd socket>'.
e.g.:
```
ctrl_interface=/var/run/hostapd
```

## 3. DHCP server: `dnsmasq`

`sudo apt-get install dnsmasq`

### 3.1 update file `/etc/dhcpcd.conf`
Open file:
`sudo nano /etc/dhcpcd.conf`

add the following lines to the end:
```
interface wlan0
static ip_address=10.1.1.10/24
denyinterfaces eth0
denyinterfaces wlan0
```

### 3.2 update `dnsmasq` config
Move the old config:
`sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig`

Create a new config file:
`sudo nano /etc/dnsmasq.conf`

**UPDATE DNS ADDRESS!!!!**
with the following lines:
```
interface=wlan0
  listen-address=::1,127.0.0.1,10.1.1.10
  no-resolv
  dhcp-option=6,{DNS1},{DNS2}
  dhcp-range=10.1.1.11,10.1.1.30,255.255.255.0,24h
```

## 4. Traffic forwarding

### 4.1 enable IP forwarding in `/etc/sysctl.conf`
`sudo nano /etc/sysctl.conf`

Uncomment the following line:
`net.ipv4.ip_forward=1`


## 5. Set wifi country, disable automatic boot
`sudo raspi-config`
- Localization Options -> WLAN Country -> US
- System Options -> Boot /Auto Login -> Console

## 6. `hostapd` is masked by default and needs to be unmasked
**If there is no need to test right away, do it later, at Step 10**
```bash
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd
```

## 7. Restart `hostapd` on startup after forwarding rules are applied
There is a problem that `hostapd` starts and hangs in a broken state on reboot.
I was able to fix it by adding the following line to `/etc/rc.local`, before `exit 0`:
`systemctl restart hostapd`

---
---
AT THIS POINT WE HAVE A WIFI ROUTER THAT HAS DHCP BUT NO INTERNET CONNECTION VIA WIFI. BELOW ARE INSTRUCTION ON ADDING OPENVPN CONNECTION.
---
---
## 8. Install OpenVPN
`sudo apt install openvpn`

## 9. Update `iptables` rules

modify and execute this `iptables.sh`
```
# Flush
iptables -t nat -F
iptables -t mangle -F
iptables -F
iptables -X

# Block All
iptables -P OUTPUT DROP
iptables -P INPUT DROP
iptables -P FORWARD DROP
ip6tables -P OUTPUT DROP
ip6tables -P INPUT DROP
ip6tables -P FORWARD DROP

#DHCP
#TODO: Expressly prohibit those to go to and from eth0
iptables -A INPUT -s 0.0.0.0 -d 255.255.255.255 -i wlan0 -p udp --dport 67:68 --sport 67:68 -j ACCEPT
iptables -A INPUT -s 10.1.1.0/24 -d 10.1.1.10 -i wlan0 -p udp --dport 67:68 --sport 67:68 -j ACCEPT
iptables -A OUTPUT -s 10.1.1.10 -d 10.1.1.0/24 -o wlan0 -p udp --dport 67:68 --sport 67:68 -j ACCEPT

#SSH
# Allow ssh via ethernet if needed.
# Via wifi might be less safe
# iptables -A INPUT -s 192.168.1.0/24 -d 192.168.1.10 -i eth0 -p tcp --dport 22 -j ACCEPT
# iptables -A OUTPUT -s 192.168.1.10 -d 192.168.1.0/24 -o eth0 -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT

#VPN
# here eth0 might not be the right name, run `ip link show`
#TODO: make VPN host the only allowed source and destination here
#??? iptables -A INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -s {VPN host} -i eth0 -p udp -j ACCEPT
iptables -I OUTPUT 1 -o eth0 -p udp --dport {VPN port} -d {VPN host} -j ACCEPT

#OpenVPN TUN Forwarding
iptables -A FORWARD -i tun0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i wlan0 -o tun0 -j ACCEPT
iptables -A FORWARD -i wlan0 -o eth0 -j DROP  
iptables -A FORWARD -i eth0 -o wlan0 -j DROP 

# TODO: Do this also??? 
#iptables -A FORWARD -i tun0 -o eth0 -j DROP  
#iptables -A FORWARD -i eth0 -o tun0 -j DROP 

iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE

# Optional debug packet logging (/var/log/kern.log)
# iptables -N logging
# iptables -A INPUT -j logging
# iptables -A OUTPUT -j logging
# iptables -A logging -m limit --limit 2/min -j LOG --log-prefix "IPTables general: " --log-level 7
# iptables -A logging -j DROP
```

Apply rules:
`sudo chmod +x iptables.sh`
`sudo ./iptables.sh`

Save rules:
`sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"`
`sudo sh -c "ip6tables-save > /etc/iptables.ipv6.nat"`

Load rules on startup - add the following line to `/etc/rc.local`, before `exit 0`:
`iptables-restore < /etc/iptables.ipv4.nat`
`ip6tables-restore < /etc/iptables.ipv6.nat`


## 10. Run OpenVPN on startup

https://www.ivpn.net/knowledgebase/linux/linux-autostart-openvpn-in-systemd-ubuntu/

Edit the config
`sudo nano /etc/default/openvpn`
-> uncomment
`AUTOSTART="all"`

Save config to
`/etc/openvpn/client.conf`
Add
`auth-user-pass {PASSFILE}`

run:
```
sudo systemctl enable openvpn@client.service
sudo systemctl daemon-reload
sudo service openvpn@client start
```

OpenVPN client logs are in : /var/log/syslog

# !!!! DONE !!!!

## P.S. If you didn't unmask hostapd earlier, unmask it
```
sudo systemctl unmask hostapd 
sudo systemctl enable hostapd 
sudo systemctl start hostapd
```


# TODO: WPA2-enterprise

https://www.bordergate.co.uk/wpa2-enteprise-access-point-with-linux/

https://android.googlesource.com/platform/external/wpa_supplicant_8/+/master/hostapd/hostapd.eap_user
---
---
---



## P.P.S To fix Realtek intermittent on-off issue:
Didn't work `sudo echo on | sudo tee /sys/bus/usb/devices/4-1/power/control`


TODO:
- disable avahi-daemon
  sudo systemctl disable avahi-daemon

- figure out why disabling dhcpcd breaks everything?
	- https://forums.raspberrypi.com/viewtopic.php?t=138161
	  https://linuxhint.com/raspberry_pi_static_ip_setup/

TODO:
- disable ssh login for unsecure nteworks
- token

TODO:
- DNS over HTTPS

- EAP
	- IEEE_802.1X
	- hostapd supported
```
		# Path for EAP server user database
		# If SQLite support is included, this can be set to "sqlite:/path/to/sqlite.db"
		# to use SQLite database instead of a text file.
		#eap_user_file=/etc/hostapd.eap_user

		# CA certificate (PEM or DER file) for EAP-TLS/PEAP/TTLS
		#ca_cert=/etc/hostapd.ca.pem

		# Server certificate (PEM or DER file) for EAP-TLS/PEAP/TTLS
		#server_cert=/etc/hostapd.server.pem
```

### Blocking all traffic except at TCP port 22

Do you have a custom shell script for firewall? Then add the following rules to your iptables based shell script:

/sbin/iptables -A INPUT -p tcp --dport 22 -j ACCEPT
/sbin/iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

First rule will accept incoming (INPUT) tcp connection on port 22 (ssh server) and second rule will send response of incoming ssh server to client (OUTPUT) from our ssh server source port 22.

### Shell script examples

However, iptables with kernel 2.4/2.6/3.x/4.x/5.x provides very powerful facility to filter rule based upon different connection states such as established or new connection etc. Here is complete small script to do this task:

```
#!/bin/sh
# Purpose: Linux Iptables Block All Incoming Traffic But Allow SSH @ TCP/22
# -------------------------------------------------------------------------------------
# My system IP/set ip address of server here
SERVER_IP="65.55.12.13"
 
# Flushing all rules
iptables -F
iptables -X
 
# Setting default filter policy
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
 
# Allow unlimited traffic on loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
 
# Allow incoming ssh only
iptables -A INPUT -p tcp -s 0/0 -d $SERVER_IP --sport 513:65535 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp -s $SERVER_IP -d 0/0 --sport 22 --dport 513:65535 -m state --state ESTABLISHED -j ACCEPT
 
# make sure nothing comes or goes out of this box
iptables -A INPUT -j DROP
iptables -A OUTPUT -j DROP
```

Raspberry pi disk encryption
https://rr-developer.github.io/LUKS-on-Raspberry-Pi/

Custom raspberry Pi image
https://opensource.com/article/21/7/custom-raspberry-pi-image

ubuntu disable  bluetooth

For Ubuntu 20.10 For this ubuntu edit /etc/bluetooth/main.conf