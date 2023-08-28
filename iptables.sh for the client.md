## 1. Install OpenVPN
`sudo apt install openvpn`

## 2. Update `iptables` rules

modify and execute this `iptables.sh`
Allow only outgoing traffic via VPN.

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

# Allow only requests on tun0.
iptables -A INPUT -i tun0 -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -o tun0 -j ACCEPT

# Also allow VPN
# here eth0 might not be the right name, run `ip link show`
#TODO: make VPN host the only allowed source and destination here
#??? iptables -A INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -s {VPN host} -i eth0 -p udp -j ACCEPT
iptables -I OUTPUT 1 -o eth0 -p udp --dport {VPN port} -d {VPN host} -j ACCEPT
```

Apply rules:
`sudo chmod +x iptables.sh`
`sudo ./iptables.sh`

### Enable persistence for iptables
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

## 3. Run OpenVPN on startup

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