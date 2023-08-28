# Raspberry Pi 2 IVPN Gateway

`Misc`  mirimir (gpg key 0x17C2E43E)



This how-to explains how to setup a Raspberry Pi 2 Model B v1.1 microcomputer as an IVPN gateway firewall/router, using Raspbian (Debian Wheezy). In addition to the Pi, you need an 8GB microSDHC card (preferably class 10) and a USB-to-ethernet adapter, which provides a second ethernet port (eth1). You connect the Pi’s WAN interface (eth0) to a LAN with Internet connectivity. An OpenVPN client establishes a VPN tunnel (tun0) to an IVPN server. The Pi forwards all traffic from devices attached to its LAN interface (eth1) through the VPN tunnel (tun0). Firewall rules allow outgoing connections on WAN (eth0) only to IVPN servers, Raspbian wheezy repository servers (for package updates) and NTP timeservers. No DNS servers are reachable via WAN (eth0) and so the IP addresses of these servers must be specified or resolved locally.

The gateway boots with no IVPN route connected, and allows no traffic to the Internet. After connecting with SSH from a local machine, you create a user-password file in /tmp, which is stored in RAM. Then you can start, stop and restart IVPN connections, with no need to reenter your username and password (until the gateway is rebooted).

Setup your Pi with a DVI monitor (perhaps via an HDMI-DVI adapter) or an HDMI TV, and a USB keyboard. It’s important to use an adequate power supply. The Pi 2 uses 600-2000mA at 5V. And some USB keyboards are power hogs. Inadequate voltage at load may lead to instability and errors. Overvoltage supplied via the micro-USB power cable will temporarily trip the polyfuse, but probably won’t cause permanent damage. However, the USB data ports bypass the polyfuse, and so voltage surges on powered USB hubs can fry the Pi. See  [http://www.raspberrypi.org/help/faqs/#powerReqs](http://www.raspberrypi.org/help/faqs/#powerReqs).

Download the Raspbian (Debian Wheezy) image archive from  [http://www.raspberrypi.org/downloads/](http://www.raspberrypi.org/downloads/)  and extract the image. Put the 8GB microSDHC card in a slot or USB adapter, and write the Raspbian wheezy image to it. Then put the card in your Pi, and attach the micro-USB power cable. At first boot, you get the raspi-config screen. Select  `Expand Filesystem`  to expand the image to fill your SD card. Then select  `Change User Password`  (default being  `raspberry`). Select  `Internationalisation Options`  to configure language, timezone and keyboard layout. Using  `Advanced Options`, change the hostname (perhaps to  `ivpngw`) and enable SSH server. Also  `Enable Boot to Desktop`, because that will facilitate setup. You can later switch back to text console, if you like. Finally, tab to  `Finish`  and let the Pi reboot. Login as as user  `pi`  with your new password.

First update the firmware, and let the Pi reboot.

```
$ sudo rpi-update

```

## Initial Setup

Configure the network interfaces. These instructions assume that the Pi WAN interface is connected to LAN <192.168.1.0/24>, and that a DHCP server at <192.168.1.1> is pushing valid DNS server(s). If your LAN IP range is different, adjust the LAN IPs in the iptables rules below accordingly. And by the way, WAN (eth0) and LAN (eth1) can’t be in the same IP range. We’ll make the Pi WAN interface static after configuring OpenVPN, and finally configure a DHCP server on the Pi LAN interface.

```
$ sudo nano /etc/network/interfaces
........................
auto lo
iface lo inet loopback

```

Now open Epiphany, browse to this how-to guide, and bookmark it. Then open LXTerminal. Now you can copy text from the guide, and paste it into the terminal, using  `Shift-Ctrl-V`.

Now update and install required packages. OK saving the default iptables rules.

```
$ sudo apt-get update
$ sudo apt-get dist-upgrade
$ sudo apt-get install host openvpn iptables-persistent

```

You must enable forwarding.

```
$ sudo cp /etc/sysctl.conf /etc/sysctl.conf.defaults
$ sudo nano /etc/sysctl.conf
......................................
kernel.printk = 3 4 1 3
net.ipv4.ip_forward=1
net.ipv4.conf.all.accept_redirects = 0
vm.swappiness=1
vm.min_free_kbytes = 8192
......................................

```

Reboot the gateway Pi

```
$ sudo reboot

```

## OpenVPN Setup

Download the latest  [OpenVPN configuration files](https://www.ivpn.net/openvpn-config)  and extract the archive to  `/home/pi`.

```
$ unzip /mnt/ivpn-openvpn-config.zip -d /home/pi/

```

Reconfigure openvpn so it doesn’t start all valid VPNs at boot. That’s necessary because IVPN requires entering username and password to connect, and the openvpn daemon doesn’t have a mechanism for prompting for entering them.

```
$ sudo nano /etc/default/openvpn
................
...
AUTOSTART="none"
...
................

```

However, there’s a workaround. At boot, create a temporary user-pass file in the  `/tmp`  tmpfs. It will be stored in RAM, and not saved to the SD card. Until you reboot the Pi, however, the credentials will remain available. Simply saving the user-pass file to the SD card is far less secure. If you like, you can encrypt the SD card using dm-crypt/LUKS with LVM2 for easy swap encryption. See  [http://www.raspberrypi.org/forums/viewtopic.php?f=29&t=102103&p=709645](http://www.raspberrypi.org/forums/viewtopic.php?f=29&t=102103&p=709645).

```
$ sudo nano /tmp/user-pass
........
username
password
........

```

We will configure iptables to block all non-VPN Internet access, except to three groups of servers: 1) IVPN servers that we want to use; 2) Raspbian wheezy repository servers, for package updates; and 3) NTP timeservers, to insure that the Pi knows the correct time. No DNS servers are reachable via WAN (eth0) and so IVPN servers must be specified by IP addresses, or resolved locally. For IVPN servers, it’s most straightforward to specify IP addresses in the config files. Choose the IVPN routes that you’ll be using, and edit their config files. Also point to  `/tmp/user-pass`, and change  `verb 3`  to  `verb 5`.

In this example, I’ll do  `IVPN-Singlehop-Netherlands`  and  `IVPN-Singlehop-Germany`. In one LXTerminal:

```
$ mkdir /home/pi/IVPN-config/edited
$ cp /home/pi/IVPN-config/IVPN-Singlehop-Netherlands.conf /home/pi/IVPN-config/edited/
$ cat /home/pi/IVPN-config/edited/IVPN-Singlehop-Netherlands.conf
...........................
...
remote gw1.nl.ivpn.net 2049
auth-user-pass
...
verb 3
...........................

```

In a second LXTerminal:

```
$ sudo host gw1.nl.ivpn.net
 => gw1.nl.ivpn.net has address 85.12.8.104

```

Back in the first LXTerminal, edit the config file, and save.

```
$ nano /home/pi/IVPN-config/edited/IVPN-Singlehop-Netherlands.conf
.............................
...
remote 85.12.8.104 2049
auth-user-pass /tmp/user-pass
...
verb 5
.............................

```

Repeat for the route  `IVPN-Singlehop-Germany`, and you should get:

```
$ nano /home/pi/IVPN-config/edited/IVPN-Singlehop-Germany.conf
.............................
...
remote 178.162.193.154 2049
auth-user-pass /tmp/user-pass
...
verb 5
.............................

```

Copy VPN credentials and selected route configs to  `/etc/openvpn`.

```
$ cd /home/pi/IVPN-config
$ sudo cp ca.crt client1.crt client1.key ta.key /etc/openvpn/
$ cd edited
$ sudo cp *.conf /etc/openvpn/

```

Remove read rights on credentials for group and other.

```
$ cd /etc/openvpn
$ sudo chmod go-r ca.crt client1.crt client1.key ta.key

```

Start  `IVPN-Singlehop-Netherlands`.

```
$ sudo service openvpn start IVPN-Singlehop-Netherlands
[ ok ] Starting virtual private network daemon: IVPN-Singlehop-Netherlands.

```

And check status.

```
$ sudo service openvpn status
 [warn] No VPN autostarted ... (warning).
 [FAIL] VPN 'IVPN-Singlehop-Germany' (non autostarted) is not running ... failed!
 [ ok ] VPN 'IVPN-Singlehop-Netherlands' (non autostarted) is running.

```

In Epiphany, browse  [https://whatismyipaddress.com/](https://whatismyipaddress.com/)

=> 85.12.8.104 [Base IP B.V.]

Stop it and start  `IVPN-Singlehop-Germany`.

```
$ sudo service openvpn stop
[ ok ] Stopping virtual private network daemon: IVPN-Singlehop-Netherlands.

```

In Epiphany, browse  [https://whatismyipaddress.com/](https://whatismyipaddress.com/)

=> 178.162.210.2 [Leaseweb Germany GmbH]

## Configure /etc/hosts

The above approach doesn’t work for Raspbian wheezy repositories and NTP (time) servers, and so we use  `/etc/hosts`. Update package lists, get the hostnames being hit, and use host to get the IP addresses.

```
$ sudo apt-get update
=> hits mirrordirector.raspbian.org, archive.raspberrypi.org and raspberrypi.collabora.com

```

Now see what NTP servers are being hit, and use host to get the IP addresses.

```
$ sudo nano /etc/ntp.conf
=> hits 0.debian.pool.ntp.org, 1.debian.pool.ntp.org, 2.debian.pool.ntp.org, and 3.debian.pool.ntp.org

```

Configure host and populate  `/etc/hosts`  with the above information.

```
$ sudo nano /etc/host.conf
....................
multi on
order hosts bind nis
....................
$ sudo nano /etc/hosts
............................................
127.0.0.1        localhost
127.0.0.1        vpngateway
5.153.225.207    mirrordirector.raspbian.org
93.93.128.211    archive.raspberrypi.org
93.93.128.230    archive.raspberrypi.org
93.93.130.39     archive.raspberrypi.org
93.93.130.214    archive.raspberrypi.org
93.93.128.223    raspberrypi.collabora.com
94.75.223.121    mirror.nl.leaseweb.net
67.198.37.16     0.debian.pool.ntp.org
82.141.152.3     0.debian.pool.ntp.org
87.195.109.207   0.debian.pool.ntp.org
95.213.132.250   0.debian.pool.ntp.org
87.230.85.6      1.debian.pool.ntp.org
92.63.212.161    1.debian.pool.ntp.org
131.234.137.24   1.debian.pool.ntp.org
188.126.88.9     1.debian.pool.ntp.org
77.245.18.26     2.debian.pool.ntp.org
83.137.98.96     2.debian.pool.ntp.org
85.214.108.169   2.debian.pool.ntp.org
193.224.65.146   2.debian.pool.ntp.org
157.7.154.29     3.debian.pool.ntp.org
176.74.25.228    3.debian.pool.ntp.org
173.230.144.109  3.debian.pool.ntp.org
193.219.61.110   3.debian.pool.ntp.org
............................................

```

## Configure iptables

Now that OpenVPN is working, configure iptables. But first make sure that the default iptables ruleset allows everything. That way, if you manage to lock yourself out, rebooting will restore access.

```
$ sudo nano /etc/iptables/rules.v4
.....................
*filter

```

You want an iptables ruleset that blocks all non-VPN connections to the Internet. It drops all input, forward and output by default, so all desired traffic must be explicitly allowed. Further, various sorts of malformed packets are dropped early, as in adrelanos' VPN-Firewall.

In the following ruleset, there are two placeholders:  `IP-of-VPN-server`  and  `port-of-VPN-server`. They come from the OpenVPN configuration file. For  `IVPN-Singlehop-Netherlands`, as we saw above, they are  `85.12.8.104`  and  `2049`. For  `IVPN-Singlehop-Germany`, they are  `178.162.193.154`  and  `2049`. You will need a line for each IVPN server that you’ll want to use.

```
$ sudo nano /etc/iptables/vpn-rules.v4
...........................................................................................
*nat

```

Verify that you can still hit repository and NTP servers.

```
$ sudo apt-get update
=> should see no errors

```

Now test IVPN-Singlehop-Netherlands and IVPN-Singlehop-Germany.

```
$ sudo service openvpn start IVPN-Singlehop-Netherlands
   [ ok ] Starting virtual private network daemon: IVPN-Singlehop-Netherlands.

```

In Epiphany, browse  [https://whatismyipaddress.com/](https://whatismyipaddress.com/).

=> 85.12.8.106 [Base IP B.V.]

Browse  [https://www.grc.com/dns/dns.htm](https://www.grc.com/dns/dns.htm)  and run standard test.

=> 85.12.5.11 is only reachable DNS server

```
$ sudo service openvpn stop
   [ ok ] Stopping virtual private network daemon: IVPN-Singlehop-Netherlands.

```

In Epiphany, browse  [https://whatismyipaddress.com/](https://whatismyipaddress.com/)

=> 178.162.210.2 [Leaseweb Germany GmbH]

Browse  [https://www.grc.com/dns/dns.htm](https://www.grc.com/dns/dns.htm)  and run standard test.

=> 178.162.193.154 is only DNS server

Repeating the above, you will get different  `inet addr`  and  `P-t-P`  values, but they will always be in  `10.9.0.0/16`  for  `IVPN-Singlehop-Netherlands`, and in  `10.20.0.0/16`  for  `IVPN-Singlehop-Germany`. The DNS server for  `IVPN-Singlehop-Netherlands`  is  `10.9.0.1`, and for  `IVPN-Singlehop-Germany`  it’s  `10.20.0.1`.

Now it’s time to reconfigure eth0 statically, because you no longer want the DNS server(s) that  `192.168.1.1`  pushes.

```
$ sudo nano /etc/network/interfaces
........................
auto lo
iface lo inet loopback

```

And now you can configure  `/etc/resolv.conf`  because DHCP won’t be changing it.

```
$ sudo nano /etc/resolv.conf
....................
domain localdomain
search localdomain
nameserver 10.9.0.1   
nameserver 10.20.0.1
....................

```

You’ll need a  `nameserver`  line for each of the IVPN routes that you’ll be using.

Now that your iptables ruleset is working, you can rename it so it loads at bootup.

```
$ sudo mv /etc/iptables/rules.v4 /etc/iptables/open-rules.v4
$ sudo mv /etc/iptables/vpn-rules.v4 /etc/iptables/rules.v4
$ sudo iptables-restore < /etc/iptables/rules.v4

```

## Add DHCP Server

Now install and configure DHCP server on eth1.

```
$ sudo apt-get update
$ sudo apt-get install isc-dhcp-server
$ sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.default
$ sudo nano /etc/dhcp/dhcpd.conf
..........................................
ddns-update-style none;
default-lease-time 600;
max-lease-time 7200;
authoritative;
subnet 192.168.2.0 netmask 255.255.255.0 {
   option subnet-mask 255.255.255.0;
   option routers 192.168.2.1;
   range 192.168.2.10 192.168.2.20;
}
..........................................

```

## Test with Workspace Client

Attach a computer to IVPN gateway Pi eth1, and test.

```
$ sudo ifconfig
   => 192.168.2.10

```

Browse  [https://whatismyipaddress.com/](https://whatismyipaddress.com/)

=> 85.12.8.105 [Base IP B.V.]

Browse  [https://www.grc.com/dns/dns.htm](https://www.grc.com/dns/dns.htm)  and run standard test.

=> 85.12.5.11 is only reachable DNS server

## Lock Down SSH Server in Gateway

Generate RSA key pair in workspace client.

```
$ ssh-keygen

```

SCP public key to IVPN gateway Pi.

```
$ scp /home/pi/.ssh/id_rsa.pub pi@192.168.2.1:/home/pi/

```

SSH to the IVPN gateway Pi.

```
 $ ssh pi@192.168.2.1

```

Working now on IVPN gateway Pi.

```
$ mkdir .ssh
$ nano /home/pi/.ssh/authorized_keys

```

Hit Ctrl-R and read in /home/pi/id_rsa.pub, and save and exit.

Now lock down sshd_config.

```
$ sudo nano /etc/ssh/sshd_config
....................................................
...
PermitRootLogin no
...
AuthorizedKeysFile	/home/pi/.ssh/authorized_keys
...
PasswordAuthentication no
...
....................................................

```

Open another LXTerminal in the workspace client to test SSH. It doesn’t matter here, because the gateway Pi is accessible, but getting locked out of a remote server can be a hassle.

```
$ ssh pi@192.168.2.1
Enter passphrase for key '/home/pi/.ssh/id_rsa': xxxxxxxxxxxxxx
```