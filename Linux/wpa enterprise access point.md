[Bordergate 2019: Creating a WPA2 Enterprise Access Point Using Linux](https://www.bordergate.co.uk/wpa2-enteprise-access-point-with-linux/)  
  
# Creating a WPA2 Enterprise Access Point Using Linux  
  
  
Most consumer Wi-Fi routers are configured to use WPA2 Personal, which has some shortcomings in terms of security.  
  
WPA2 Enterprise addresses these shortcomings by allowing individual username and passwords for each client, in addition to allowing for certificate-based authentication allowing clients to verify the authenticity of the access point.  
  
This guide shows how to setup a Fedora 29 Linux system with an AWUS036NH wireless antenna to act as a secure wireless access point. Hostapd and FreeRADIUS will be used to achieve this.  
  
#### Check the wireless card is detected by the OS  
  
Check the device is recognised using lsusb:  
  
```  
lsusb | grep -i wireless  
---  
Bus 001 Device 006: ID 148f:3070 Ralink Technology, Corp. RT2870/RT3070 Wireless Adapter  
```  
  
Check the adapters MAC address (we will need this later):  
```  
ifconfig  
---  
wlp0s21f0u1: flags=4099<up,broadcast,multicast> mtu 1500  
ether 96:fe:34:a4:fa:4b txqueuelen 1000 (Ethernet)  
RX packets 0 bytes 0 (0.0 B)  
RX errors 0 dropped 0 overruns 0 frame 0  
TX packets 0 bytes 0 (0.0 B)  
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0  
```  
  
Configure NetworkManager to ignore the device, based on the MAC address:  
```  
vi /etc/NetworkManager/NetworkManager.conf  
---  
[keyfile]  
unmanaged-devices=mac:96:fe:34:a4:fa:4b  
```  
  
Restart NetworkManager for the change to take effect:  
```  
systemctl restart NetworkManager.service  
```  
  
#### Setting up Hostapd  
  
Install the necessary packages:  
```  
dnf install hostapd freeradius iptables  
```  
  
Start by creating certificates required for authentication:  
```  
cd /etc/raddb/certs/  
rm -f *.pem *.der *.csr *.crt *.key *.p12 serial* index.txt*  
./bootstrap  
```  
  
Copy the certificates to the hostapd directory to prevent selinux triggering:  
```  
cp /etc/raddb/certs/* /etc/hostapd/certs/  
```  
  
Modify the hostapd configuration file, including the below parameters:  
```  
vi /etc/hostapd/hostapd.conf  
---  
interface=wlp0s21f0u1  
driver=nl80211  
ssid=WLAN1  
channel=6  
auth_algs=1  
eap_server=1  
ieee8021x=1  
eapol_version=2  
wpa=2  
wpa_key_mgmt=WPA-EAP  
wpa_pairwise=TKIP  
rsn_pairwise=CCMP  
eap_user_file=/etc/hostapd/hostapd.eap_user  
ca_cert=/etc/hostapd/certs/ca.pem  
server_cert=/etc/hostapd/certs/server.pem  
private_key=/etc/hostapd/certs/server.key  
private_key_passwd=whatever  
dh_file=/etc/hostapd/certs/dh  
logger_syslog=-1  
logger_syslog_level=2  
logger_stdout=-1  
logger_stdout_level=2  
ctrl_interface=/var/run/hostapd  
ctrl_interface_group=0  
hw_mode=g  
ieee80211n=1  
wme_enabled=1  
```  
  
#### **Configure Users**  
```  
vi /etc/hostapd/hostapd.eap_user  
---  
* PEAP,TTLS  
"testaccount1" MSCHAPV2 "SuperSecretPassword1" [2]  
"testaccount2" MSCHAPV2 " SuperSecretPassword2" [2]  
```  
  
#### Install DNSMasq  
  
DNSMasq provides DHCP services for the access point.  
```  
vi /etc/dnsmasq.conf  
---  
interface=wlp0s21f0u1  
dhcp-range=192.168.2.4,192.168.2.50,255.255.255.0,24h  
```  
```  
systemctl start dnsmasq.service  
```  
#### Enable IP Forwarding  
  
Enabling IP forwarding allows traffic from the Wifi adapter to be forwarded through the systems default gateway:  
```  
sysctl net.ipv4.ip_forward=1  
```  
Make the change permanent by changing /etc/sysctl.conf:  
```  
vi /etc/sysctl.conf  
---  
net.ipv4.ip_forward=1  
```  
Ensure network address translation is applied to traffic leaving the external interface (in this case enp2s0):  
```  
iptables -t nat -A POSTROUTING -o enp2s0 -j MASQUERADE  
```  
Save the rules to run on reboot:  
```  
iptables-save > /etc/sysconfig/iptables  
```  
#### Set services to start on boot  
```  
systemctl enable hostapd.service  
systemctl enable dnsmasq.service  
systemctl enable iptables  
```  
That’s it! You should now be able to connect to the wireless access point. You will be prompted to verify the server certificate the first time you connect, and then for the username and password previously configured.