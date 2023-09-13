It's possible to perform a debian update fully offline.  
For that we need to get a list of deb packages up front, by using `apt download`.  
  
## 0. Disable Bluetooth and wifi  
See `Raspi linux ROUTER 0`, `Raspi linux ROUTER 1 Appdx1 - eth1`.  
  
  
## 1. Choose a mirror and set up network security  
  
* Use a list of mirrors to find a good mirror, ideally `https` for best security.  
Optionally switch to a mirror (consider https mirrors):  
`sudo nano /etc/apt/sources.list`  
listed here: https://www.raspbian.org/RaspbianMirrors  
  
* Add mirror IP address to the `/etc/apt/sources`. Don't forget https!!!  
  
* Add mirror's IP address and hostname to the `/etc/hosts` (fully local DNS resolution)  
Also, to avoid the system falling back to http on archive.raspberrypi.org, set  
`127.0.0.1 archive.raspberrypi.org` in /etc/hosts.  
  
* Apply a restrictive iptables config  
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
  
# Allow only outgoing requests for your IP for software update.  
iptables -A INPUT -p tcp -m state -s <mirror_hostname IP> --state ESTABLISHED,RELATED -j ACCEPT  
iptables -A OUTPUT -p tcp -d <mirror_hostname IP> --dport 443 -j ACCEPT  
```  
  
* In order for TLS certificates to work local date and time should be adjusted accordingly.  
	- Udate timezone  
	`timedatectl list-timezones`  
	`sudo timedatectl set-timezone <your_time_zone>`  
	- Update datetime  
	`sudo date -s “YYYY-MM-DD HH:MM:SS”`  
  
## 2. Connect to network and download packages  
  
* Now it's safe to connect the network (`eth` preferred) and download packages.  
  
* Check the certificate for the host first.  
`openssl s_client -connect <mirror_hostname>:443`  
  
* Updates list of packages from selected mirror. If (1) was set up correctly, only selected mirror should be used.  
`sudo apt-get update`  
  
* Determine the full list of dependencies:  
`sudo apt-get install <pkg1> <pkg2> <pkg3> ...`, but once it shows you the list of packages. choose "no".  
This is needed because `apt download` by itself doesn't download missing dependencies, and you need all of those to install packages offline, not just the packages themselves.  
  
* Run download on all of the packages.  
`sudo apt-get download <dep1> <dep2> <dep3> ... <pkg1> <pkg2> <pkg3> ...`  
  
* Disconnect network, move the *.deb packages to a portable media to reuse later.  
  
* For convenience, determine the order of installation of packages and create an installer script, running `dpkg -i <pkg>` on those packages in the right order. ("install-packages.sh")  
  
## 2.1 Packages from archive that are not in the mirror  
  
* Some packages don't exist on the mirror, and `apt` will try to download those from the `archive.raspberrypi.org`. While archive supports https, `apt` will use http regardless. With the above networking setup, `apt` will fail to connect to archive, and you will see error messages like:  
```  
E: Failed to fetch http://archive.raspberrypi.org/debian/pool/main/c/cmake/cmake_3.18.4-2%2brpt1%2brpi1%2bdeb11u1_armhf.deb  Could not connect to archive.raspberrypi.  
org:80 (127.0.0.1), connection timed out  
E: Failed to fetch http://archive.raspberrypi.org/debian/pool/main/c/cmake/cmake-data_3.18.4-2%2brpt1%2brpi1%2bdeb11u1_all.deb  Unable to connect to archive.raspberry  
pi.org:http:  
E: Failed to fetch http://archive.raspberrypi.org/debian/pool/main/a/alsa-lib/libasound2-dev_1.2.4-1.1%2brpt2_armhf.deb  Unable to connect to archive.raspberrypi.org:  
http:  
```  
If you copy the errors and replace `http` with `https`, the links from the above error messages can be used to manually retrieve missing packages in a secure fashion via TLS.  
  
## 3. Install packages. Prevent updates   
  
* On a new machine, run the script ("install-packages.sh") with `sudo` or run `sudo dpkg -i <pkg>` for each deb package on the machine you want to install those on. Internet connection not required.  
  
* To prevent updates from happening at all  
	- comment everything out from `sudo nano /etc/apt/sources.list`  
	- add the following lines to `/etc/hosts`  
	 `127.0.0.1 archive.raspberrypi.org`  
	 `127.0.0.1 raspbian.raspberrypi.org`  
  
