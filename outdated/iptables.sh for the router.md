#!/bin/bash  
  
  
# https://www.youtube.com/watch?v=xFficDCEv3c  
  
```  
# Flush  
iptables -t nat -F  
iptables -t mangle -F  
iptables -F  
iptables -X  
  
  
# B
lock All  
iptables -P OUTPUT DROP  
iptables -P INPUT DROP  
iptables -P FORWARD DROP  
ip6tables -P OUTPUT DROP  
ip6tables -P INPUT DROP  
ip6tables -P FORWARD DROP  
  
  
# allow Localhost  
iptables -A INPUT -i lo -j ACCEPT  
iptables -A OUTPUT -o lo -j ACCEPT  
  
  
# Make sure you can communicate with any DHCP server  
# -d and -s should be chenaged if localhost is a dhcp server  
iptables -A OUTPUT -d 255.255.255.255 -j ACCEPT  
iptables -A INPUT -s 255.255.255.255 -j ACCEPT  
  
  
# Make sure that you can communicate within your own network  
iptables -A INPUT -s 10.0.1.0/24 -d 10.0.1.0/24 -j ACCEPT  
iptables -A OUTPUT -s 10.0.1.0/24 -d 10.0.1.0/24 -j ACCEPT  
  
  
# Allow established sessions to receive traffic:  
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT  
  
  
# Allow TUN  
iptables -A INPUT -i tun+ -j ACCEPT  
iptables -A FORWARD -i tun+ -j ACCEPT  
iptables -A FORWARD -o tun+ -j ACCEPT  
iptables -t nat -A POSTROUTING -o tun+ -j MASQUERADE  
iptables -A OUTPUT -o tun+ -j ACCEPT  
  
  
# allow VPN connection on port 1195  
iptables -I OUTPUT 1 -p udp --destination-port 1195 -m comment --comment "Allow VPN connection" -j ACCEPT  
  
  
# Redundant?  
# Block All  
iptables -A OUTPUT -j DROP  
iptables -A INPUT -j DROP  
iptables -A FORWARD -j DROP  
  
  
# Log all dropped packages, debug only.  
  
  
iptables -N logging  
iptables -A INPUT -j logging  
iptables -A OUTPUT -j logging  
iptables -A logging -m limit --limit 2/min -j LOG --log-prefix "IPTables general: " --log-level 7  
iptables -A logging -j DROP  
  
  
echo "saving"  
iptables-save > /etc/iptables.rules  
echo "done"  
#echo 'openVPN - Rules successfully applied, we start "watch" to verify IPtables in realtime (you can cancel it as usual CTRL + c)'  
#sleep 3  
#watch -n 0 "sudo iptables -nvL"  
```