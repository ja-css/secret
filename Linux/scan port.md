1.
nc [-options] host-ip-adress port-number
$ nc -zvw10 192.168.0.1 22

2. 
nmap [-options] [IP or Hostname] [-p] [PortNumber]
nmap 192.168.0.1 -p 22
nmap 192.168.0.1 -p 22

3. 
$ telnet [IP or Hostname] [PortNumber]

4. 
echo > /dev/tcp/[host]/[port] && echo "Port is open"
echo > /dev/udp/[host]/[port] && echo "Port is open"

5.  
netstat -tuplen