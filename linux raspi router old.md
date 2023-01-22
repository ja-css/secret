

# Preparation

**!!! The following doesn't work on 64 BIT OS!!!**
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
`sudo dd if=/dev/zero of=/dev/sda bs=512 count=1`
Unmount, unplug, replug.
Then:
**Make sure you're witing an *.img file and not *.img.xz**
**Extract *.img.xz to get *.img**
`sudo dd bs=1M if=2022-04-04-raspios-bullseye-arm64-lite.img of=/dev/sda conv=fsync status=progress`

After installation, update OS:
`sudo apt-get update`
`sudo apt-get upgrade`

## 2. WiFi hotspot: `hostapd`
All in one:
`sudo apt install hostapd dnsmasq iptables openvpn dnsutils`

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
```

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

with the following lines:
```
interface=wlan0
  listen-address=::1,127.0.0.1,10.1.1.10
  no-resolv
  server={DNS1}
  server={DNS2}
  dhcp-range=10.1.1.11,10.1.1.30,255.255.255.0,24h
```

## 4. Traffic forwarding

# TODO : need to try bridge to see if that solves UDP issue

### 4.1 enable IP forwarding in `/etc/sysctl.conf`
`sudo nano /etc/sysctl.conf`

Uncomment the following line:
`net.ipv4.ip_forward=1`


## 5. Set wifi country, disable automatic boot
`sudo raspi-config`
- Localization Options -> WLAN Country -> US
- System Options -> Boot /Auto Login -> Console

## 6. `hostapd` is masked by default and needs to be unmasked
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

# allow Localhost
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Make sure clients can communicate with local DHCP server
iptables -A OUTPUT -s 255.255.255.255 -j ACCEPT
iptables -A INPUT -d 255.255.255.255 -j ACCEPT

# Make sure that you can communicate within your own network
iptables -A INPUT -s 192.168.1.0/24 -d 192.168.1.0/24 -j ACCEPT
iptables -A OUTPUT -s 192.168.1.0/24 -d 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -s 10.1.1.0/24 -d 10.1.1.0/24 -j ACCEPT
iptables -A OUTPUT -s 10.1.1.0/24 -d 10.1.1.0/24 -j ACCEPT

# Allow established sessions to receive traffic:
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#DNS
iptables -A OUTPUT -o eth0 -p udp --dport 53 -d {DNS1} -j ACCEPT
iptables -A OUTPUT -o eth0 -p udp --dport 53 -d {DNS2} -j ACCEPT

#VPN
sudo iptables -I OUTPUT 1 -o eth0 -p udp --dport {VPN port} -d {VPN host} -j ACCEPT

# Allow TUN Forwarding
#why this line?
iptables -A INPUT -i tun+ -j ACCEPT
iptables -A FORWARD -i tun0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i wlan0 -o tun0 -j ACCEPT
iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
#why this line?
iptables -A OUTPUT -o tun+ -j ACCEPT

# Optional debug packet logging (/var/log/kern.log)
# iptables -N logging
# iptables -A INPUT -j logging
# iptables -A OUTPUT -j logging
# iptables -A logging -m limit --limit 2/min -j LOG --log-prefix "IPTables general: " --log-level 7
# iptables -A logging -j DROP
```

Apply rules:
`chmod +x iptables.sh`
`sudo ./iptables.sh`

Save rules:
`sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"`

Load rules on startup - add the following line to `/etc/rc.local`, before `exit 0`:
`iptables-restore < /etc/iptables.ipv4.nat`


## 10. Run OpenVPN on startup

Run with `nohup` on startup: add the following line to `rc.local`, before `exit 0`
`nohup openvpn --config FILENAME.ovpn --auth-user-pass auth.txt &`
`auth.txt` file content:
> username  
> password

If you didn't unmask hostapd earlier, unmask it
```
sudo systemctl unmask hostapd 
sudo systemctl enable hostapd 
sudo systemctl start hostapd
```

## 12. Disable Bluetooth completely
[Source](https://di-marco.net/blog/it/2020-04-18-tips-disabling_bluetooth_on_raspberry_pi/#disable-bluetooth-completely)

#### Uninstall  `BlueZ`  and related packages

If  `Bluetooth`  is not required at all, uninstall  `Bluetooth`  stack. It makes  `Bluetooth`  unavailable even if external  `Bluetooth`  adapter is plugged in.

This will also remove bluetooth  firmware itself.
```
sudo apt-get purge bluez -y
sudo apt-get autoremove -y
```