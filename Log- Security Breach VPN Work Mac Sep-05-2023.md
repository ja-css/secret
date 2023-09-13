  
**person1**  
Hello guys, I was learning some random stuff about openssl, and found commands to show server's certificates.    
I ran  `openssl s_client -connect` `[www.google.com:443](http://www.google.com:443/)`  and it showed me some strange certificate and error  `verify error:num=20:unable to get local issuer certificate`.    
After that it started showing me a completely different certificate.    
Does it look fishy to you?Command outputs and additional details inside.  
  
---  
  
**person1**  
Previous output with error:  
```  
$ echo -n | openssl s_client -connect www.google.com:443  
  
CONNECTED(00000003)  
depth=1 /C=ZA/O=Thawte Consulting (Pty) Ltd./CN=Thawte SGC CA  
verify error:num=20:unable to get local issuer certificate  
verify return:0  
---  
Certificate chain  
 0 s:/C=US/ST=California/L=Mountain View/O=Google Inc/CN=www.google.com  
   i:/C=ZA/O=Thawte Consulting (Pty) Ltd./CN=Thawte SGC CA  
 1 s:/C=ZA/O=Thawte Consulting (Pty) Ltd./CN=Thawte SGC CA  
   i:/C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority  
---  
Server certificate  
-----BEGIN CERTIFICATE-----  
MIIDITCCAoqgAwIBAgIQL9+89q6RUm0PmqPfQDQ+mjANBgkqhkiG9w0BAQUFADBM  
MQswCQYDVQQGEwJaQTElMCMGA1UEChMcVGhhd3RlIENvbnN1bHRpbmcgKFB0eSkg  
THRkLjEWMBQGA1UEAxMNVGhhd3RlIFNHQyBDQTAeFw0wOTEyMTgwMDAwMDBaFw0x  
MTEyMTgyMzU5NTlaMGgxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlh  
MRYwFAYDVQQHFA1Nb3VudGFpbiBWaWV3MRMwEQYDVQQKFApHb29nbGUgSW5jMRcw  
FQYDVQQDFA53d3cuZ29vZ2xlLmNvbTCBnzANBgkqhkiG9w0BAQEFAAOBjQAwgYkC  
gYEA6PmGD5D6htffvXImttdEAoN4c9kCKO+IRTn7EOh8rqk41XXGOOsKFQebg+jN  
gtXj9xVoRaELGYW84u+E593y17iYwqG7tcFR39SDAqc9BkJb4SLD3muFXxzW2k6L  
05vuuWciKh0R73mkszeK9P4Y/bz5RiNQl/Os/CRGK1w7t0UCAwEAAaOB5zCB5DAM  
BgNVHRMBAf8EAjAAMDYGA1UdHwQvMC0wK6ApoCeGJWh0dHA6Ly9jcmwudGhhd3Rl  
LmNvbS9UaGF3dGVTR0NDQS5jcmwwKAYDVR0lBCEwHwYIKwYBBQUHAwEGCCsGAQUF  
BwMCBglghkgBhvhCBAEwcgYIKwYBBQUHAQEEZjBkMCIGCCsGAQUFBzABhhZodHRw  
Oi8vb2NzcC50aGF3dGUuY29tMD4GCCsGAQUFBzAChjJodHRwOi8vd3d3LnRoYXd0  
ZS5jb20vcmVwb3NpdG9yeS9UaGF3dGVfU0dDX0NBLmNydDANBgkqhkiG9w0BAQUF  
AAOBgQCfQ89bxFApsb/isJr/aiEdLRLDLE5a+RLizrmCUi3nHX4adpaQedEkUjh5  
u2ONgJd8IyAPkU0Wueru9G2Jysa9zCRo1kNbzipYvzwY4OA8Ys+WAi0oR1A04Se6  
z5nRUP8pJcA2NhUzUnC+MY+f6H/nEQyNv4SgQhqAibAxWEEHXw==  
-----END CERTIFICATE-----  
subject=/C=US/ST=California/L=Mountain View/O=Google Inc/CN=www.google.com  
issuer=/C=ZA/O=Thawte Consulting (Pty) Ltd./CN=Thawte SGC CA  
---  
No client certificate CA names sent  
---  
SSL handshake has read 1777 bytes and written 316 bytes  
---  
New, TLSv1/SSLv3, Cipher is AES256-SHA  
Server public key is 1024 bit  
Compression: NONE  
Expansion: NONE  
SSL-Session:  
    Protocol  : TLSv1  
    Cipher    : AES256-SHA  
    Session-ID: 748E2B5FEFF9EA065DA2F04A06FBF456502F3E64DF1B4FF054F54817C473270C  
    Session-ID-ctx:   
    Master-Key: C4284AE7D76421F782A822B3780FA9677A726A25E1258160CA30D346D65C5F4049DA3D10A41F3FA4816DD9606197FAE5  
    Key-Arg   : None  
    Start Time: 1266259321  
    Timeout   : 300 (sec)  
    Verify return code: 20 (unable to get local issuer certificate)  
---  
```  
  
---  
  
**person1**  
Current output:  
```  
$ echo -n | openssl s_client -connect www.google.com:443  
  
CONNECTED(00000005)  
depth=2 C = US, O = Google Trust Services LLC, CN = GTS Root R1  
verify return:1  
depth=1 C = US, O = Google Trust Services LLC, CN = GTS CA 1C3  
verify return:1  
depth=0 CN = www.google.com  
verify return:1  
---  
Certificate chain  
 0 s:CN = www.google.com  
  i:C = US, O = Google Trust Services LLC, CN = GTS CA 1C3  
  a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256  
  v:NotBefore: Aug 7 12:22:44 2023 GMT; NotAfter: Oct 30 12:22:43 2023 GMT  
 1 s:C = US, O = Google Trust Services LLC, CN = GTS CA 1C3  
  i:C = US, O = Google Trust Services LLC, CN = GTS Root R1  
  a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256  
  v:NotBefore: Aug 13 00:00:42 2020 GMT; NotAfter: Sep 30 00:00:42 2027 GMT  
 2 s:C = US, O = Google Trust Services LLC, CN = GTS Root R1  
  i:C = BE, O = GlobalSign nv-sa, OU = Root CA, CN = GlobalSign Root CA  
  a:PKEY: rsaEncryption, 4096 (bit); sigalg: RSA-SHA256  
  v:NotBefore: Jun 19 00:00:42 2020 GMT; NotAfter: Jan 28 00:00:42 2028 GMT  
---  
Server certificate  
-----BEGIN CERTIFICATE-----  
MIIEijCCA3KgAwIBAgIRAP6UXaJjZ3lCCpQiSXBoswYwDQYJKoZIhvcNAQELBQAw  
RjELMAkGA1UEBhMCVVMxIjAgBgNVBAoTGUdvb2dsZSBUcnVzdCBTZXJ2aWNlcyBM  
TEMxEzARBgNVBAMTCkdUUyBDQSAxQzMwHhcNMjMwODA3MTIyMjQ0WhcNMjMxMDMw  
MTIyMjQzWjAZMRcwFQYDVQQDEw53d3cuZ29vZ2xlLmNvbTBZMBMGByqGSM49AgEG  
CCqGSM49AwEHA0IABMFjofb8DuX9wfDenqZ359yexlwyliTz8LgkrLgOYTD947IA  
aJXImz58d7jbZ8RN2JKfOAOO6vFm0jlwCbVV5dejggJpMIICZTAOBgNVHQ8BAf8E  
BAMCB4AwEwYDVR0lBAwwCgYIKwYBBQUHAwEwDAYDVR0TAQH/BAIwADAdBgNVHQ4E  
FgQU1HJkr7hnyclXkdIoOwXnHb8sudswHwYDVR0jBBgwFoAUinR/r4XN7pXNPZzQ  
4kYU83E1HScwagYIKwYBBQUHAQEEXjBcMCcGCCsGAQUFBzABhhtodHRwOi8vb2Nz  
cC5wa2kuZ29vZy9ndHMxYzMwMQYIKwYBBQUHMAKGJWh0dHA6Ly9wa2kuZ29vZy9y  
ZXBvL2NlcnRzL2d0czFjMy5kZXIwGQYDVR0RBBIwEIIOd3d3Lmdvb2dsZS5jb20w  
IQYDVR0gBBowGDAIBgZngQwBAgEwDAYKKwYBBAHWeQIFAzA8BgNVHR8ENTAzMDGg  
L6AthitodHRwOi8vY3Jscy5wa2kuZ29vZy9ndHMxYzMvUXFGeGJpOU00OGMuY3Js  
MIIBBgYKKwYBBAHWeQIEAgSB9wSB9ADyAHcAtz77JN+cTbp18jnFulj0bF38Qs96  
nzXEnh0JgSXttJkAAAGJ0CoTggAABAMASDBGAiEAu+VP1vaHuGcm4y3Wq5RR7lVw  
g2J/K7eSk3PfW2KUKWcCIQDTMFx87yUITvF2VtglI0VpLkpxaeisqCSYD9ltXJiN  
QwB3ALNzdwfhhFD4Y4bWBancEQlKeS2xZwwLh9zwAw55NqWaAAABidAqE4MAAAQD  
AEgwRgIhAO3zgSKR16p4jDM5LngyEG06fjF5zymB8IQ7caZyrOPDAiEAvFmIkN7O  
gfJBf9hsHratSnAZgsruCnhGx59rJnyeEq4wDQYJKoZIhvcNAQELBQADggEBANb8  
/A5Mw4hu6T/vtIjvj27IbZRuI0qp4jysP5gA6IwYpq1EPnkpK+zqWU157kAVxsAX  
/zJ3iSwENKP0MQOa5RrpCcstctRr4bTk80MyT4wAbmDS4hJQW8I1QgxtCm8ZaOso  
fG8xXr7G2CbnwPd5iFVi3xCzgEK2PPL0eOC4cAINIybTp8qfBWPZvWQMRpg/qYle  
rnGmZjVFzHnWhwkvDaN9he0/bcSGcBudj9Ox4kvQgv4cBRqCRdXsFhLHpvZcS7lb  
0jbT/ZNhpOC3HSMHQA7FMsN6gKc23UvrKLBGHa7W7HbDj55fXfH2++MIW7TMS5pB  
BKbGTHFRicLM3wRgoeA=  
-----END CERTIFICATE-----  
subject=CN = www.google.com  
issuer=C = US, O = Google Trust Services LLC, CN = GTS CA 1C3  
---  
No client certificate CA names sent  
Peer signing digest: SHA256  
Peer signature type: ECDSA  
Server Temp Key: X25519, 253 bits  
---  
SSL handshake has read 4297 bytes and written 396 bytes  
Verification: OK  
---  
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384  
Server public key is 256 bit  
This TLS version forbids renegotiation.  
Compression: NONE  
Expansion: NONE  
No ALPN negotiated  
Early data was not sent  
Verify return code: 0 (ok)  
---  
DONE  
```  
  
---  
  
**person1**  
Found something else: insecure DNS  `nslookup`  returns some strange IP for  `[www.google.com](http://www.google.com/)`    
and whois doesn't think it's google.    
```  
% nslookup -debug www.google.com       
Server:		127.0.0.1  
Address:	127.0.0.1#53  
  
------------  
  QUESTIONS:  
	www.google.com, type = A, class = IN  
  ANSWERS:  
  -> www.google.com  
	internet address = 212.115.105.72  
	ttl = 60  
  AUTHORITY RECORDS:  
  ADDITIONAL RECORDS:  
------------  
Non-authoritative answer:  
Name:	www.google.com  
Address: 212.115.105.72  
```  
  
---  
  
**person1**  
While DNS over HTTPS shows a different IP:  
```  
% curl -H 'accept: application/dns-json' '[https://cloudflare-dns.com/dns-query?name=www.google.com](https://cloudflare-dns.com/dns-query?name=www.google.com)'                   
{"Status":0,"TC":false,"RD":true,"RA":true,"AD":false,"CD":false,"Question":[{"name":"www.google.com","type":1}],"Answer":[{"name":"www.google.com","type":1,"TTL":224,"data":"142.251.211.228"}]}%  
```  
  
---  
  
**person1**  
whois  
```  
% whois 212.115.105.72  
% IANA WHOIS server  
% for more information on IANA, visit [http://www.iana.org](http://www.iana.org/)  
% This query returned 1 object  
  
refer:    whois.ripe.net  
  
inetnum:   212.0.0.0 - 212.255.255.255  
organisation: RIPE NCC  
status:    ALLOCATED  
  
whois:    whois.ripe.net  
  
changed:   1997-10  
source:    IANA  
  
# whois.ripe.net  
  
inetnum:    212.115.104.0 - 212.115.105.255  
netname:    BANDITO-NETWORKS  
country:    NL  
admin-c:    SB27266-RIPE  
tech-c:     SB27266-RIPE  
status:     ASSIGNED PA  
mnt-by:     mnt-nl-xsusenet-1  
created:    2021-03-08T15:42:25Z  
last-modified: 2021-03-08T15:42:25Z  
source:     RIPE  
  
role:      Savvas Bout  
address:    1200 Brickell Ave Ste 1950  
address:    33131  
address:    Miami  
address:    UNITED STATES  
phone:     +31 (085) 5301 5703  
nic-hdl:    SB27266-RIPE  
mnt-by:     mnt-nl-xsusenet-1  
created:    2019-11-12T10:58:55Z  
last-modified: 2021-03-08T13:09:25Z  
source:     RIPE # Filtered  
  
% Information related to '212.115.104.0/22AS44016'  
  
route:     212.115.104.0/22  
origin:     AS44016  
mnt-by:     MNT-PREFIXX  
created:    2021-12-12T13:34:16Z  
last-modified: 2021-12-12T13:34:16Z  
source:     RIPE  
  
% This query was served by the RIPE Database Query Service version 1.107 (ABERDEEN)  
```  
  
---  
  
**person1**  
I mean, REALLY strange:  
```  
person1@computer out % nslookup -debug www.google.com 1.1.1.1                                
Server:		1.1.1.1  
Address:	1.1.1.1#53  
  
------------  
  QUESTIONS:  
	www.google.com, type = A, class = IN  
  ANSWERS:  
  -> www.google.com  
	internet address = 212.115.105.72  
	ttl = 60  
  AUTHORITY RECORDS:  
  ADDITIONAL RECORDS:  
------------  
Non-authoritative answer:  
Name:	www.google.com  
Address: 212.115.105.72  
  
person1@computer out % nslookup -debug www.google.com 8.8.8.8  
Server:		8.8.8.8  
Address:	8.8.8.8#53  
  
------------  
  QUESTIONS:  
	www.google.com, type = A, class = IN  
  ANSWERS:  
  -> www.google.com  
	internet address = 212.115.105.72  
	ttl = 60  
  AUTHORITY RECORDS:  
  ADDITIONAL RECORDS:  
------------  
Non-authoritative answer:  
Name:	www.google.com  
Address: 212.115.105.72  
  
person1@computer out % nslookup -debug www.google.com 77.88.8.8  
Server:		77.88.8.8  
Address:	77.88.8.8#53  
  
------------  
  QUESTIONS:  
	www.google.com, type = A, class = IN  
  ANSWERS:  
  -> www.google.com  
	internet address = 212.115.105.72  
	ttl = 60  
  AUTHORITY RECORDS:  
  ADDITIONAL RECORDS:  
------------  
Non-authoritative answer:  
Name:	www.google.com  
Address: 212.115.105.72  
```  
  
---  
  
**person1**  
I'll disconnect my network until tomorrow.  
  
---  
  
**person2**  
There is some interception of DNS by the mobile device management tool we are using  
To secure all laptops  
  
---  
  
**person3**  
Even without umbrella, macOS runs an interceptor and caching of DNS. Depending on your ISP, there may be an intercept and cache…  
With that said, to understand what is going on with DNS use dig, not nslookup.  
  
---  
  
**person4**  
We've also found that people at company tend to use all manner of strange VPN software, either on their laptop or indirectly through the local network, that also intercept connections, inject their own certificates, etc. You might be getting this as a result of something trying to intercept the initial connection, but the failure of the cert parsing means that TLS is workingAnother general note about openssl s_client: you need to use '-hostname  [www.google.com](http://www.google.com/)' in addition to actually make it set the correct SNI field in the TLS Hello packet. If you don't, it's just an specified connection to an IP address, which may not request the right certificate from the virtual host on the other side.So, always prefer something like this:  `openssl s_client -connect` `[www.google.com:443](http://www.google.com:443/)` `-servername` `[www.google.com](http://www.google.com/)`  . I know, it makes no sense that it's not the default behavior. I don't think that's what happened necessarily with  _your_  IP, but it's important to do often in other cases. You'll get the following error in that case if there's a mismatch:    
```  
% openssl s_client -connect api.tryotter.com:443 -servername www.google.com  
CONNECTED(00000005)  
8054E24BF87F0000:error:0A000410:SSL routines:ssl3_read_bytes:sslv3 alert handshake failure:ssl/record/rec_layer_s3.c:1586:SSL alert number 40  
---  
no peer certificate available  
```  
  
---  
  
**person3**  
Examples of dig:    
1.  using “local” DNS…which runs through umbrella…  
```  
% dig google.com  
  
; <<>> DiG 9.10.6 <<>> google.com  
;; global options: +cmd  
;; Got answer:  
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59921  
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags:; udp: 4096  
;; QUESTION SECTION:  
;google.com.			IN	A  
  
;; ANSWER SECTION:  
google.com.		300	IN	A	142.250.65.174  
  
;; Query time: 28 msec  
;; SERVER: 127.0.0.1#53(127.0.0.1)  
;; WHEN: Tue Sep 05 08:07:47 EDT 2023  
;; MSG SIZE  rcvd: 55  
  
2) Dig direct to google DNS, which will skip umbrella:    
  
% dig @8.8.8.8 www.google.com  
  
; <<>> DiG 9.10.6 <<>> @8.8.8.8 www.google.com  
; (1 server found)  
;; global options: +cmd  
;; Got answer:  
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50332  
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags:; udp: 512  
;; QUESTION SECTION:  
;www.google.com.			IN	A  
  
;; ANSWER SECTION:  
www.google.com.		299	IN	A	142.250.81.228  
  
;; Query time: 75 msec  
;; SERVER: 8.8.8.8#53(8.8.8.8)  
;; WHEN: Tue Sep 05 08:33:29 EDT 2023  
;; MSG SIZE  rcvd: 59   
  
…notice the IP address is different…    
3) and again via local, only a few minutes later…    
  
% dig www.google.com   
  
; <<>> DiG 9.10.6 <<>> www.google.com  
;; global options: +cmd  
;; Got answer:  
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19712  
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags:; udp: 4096  
;; QUESTION SECTION:  
;www.google.com.			IN	A  
  
;; ANSWER SECTION:  
www.google.com.		230	IN	A	142.251.40.132  
  
;; Query time: 13 msec  
;; SERVER: 127.0.0.1#53(127.0.0.1)  
;; WHEN: Tue Sep 05 08:34:41 EDT 2023  
;; MSG SIZE  rcvd: 59  
```  
…notice the IP address is changed, again.What is happening? Well,  [google.com](http://google.com/)  has multiple A records. Lots of them. Which A record is returned will depend on your location (as perceived by the DNS server) and (likely) the load/availability of any given server to answer your request. Welcome to DNS based load distribution.[www.google.com](http://www.google.com/)  
  
---  
  
**person1**  
@person2 you wrote:    
> There is some interception of DNS by the mobile device management tool we are using  
  
is this your address? 212.115.105.72  
  
---  
  
**person3**  
No, intercept ==> Umbrella. When you use the default local DNS (e.g. at 127.0.0.1), it will run through Umbrella.  
Umbrella then makes the DNS call (through secure channels to a dedicated cloud service) and returns your result.  
  
---  
  
**person1**  
Just ran another test.    
I am using ExpressVPN, and  `212.115.105.72`  is returned only when I'm connected via Express VPN. Which probably means that your Umbrella thing doesn't work.  
  
---  
  
**person3**  
Some VPNs don’t play nice with Umbrella. Why are you using a VPN?  
  
---  
  
**person1**  
For better security.  
just a second ago, the default reply changed, but direct access still returns that strange IP.  
```  
person1@computer out % nslookup -debug www.google.com   
Server:		127.0.0.1  
Address:	127.0.0.1#53  
  
------------  
  QUESTIONS:  
	www.google.com, type = A, class = IN  
  ANSWERS:  
  -> www.google.com  
	internet address = 142.250.217.100  
	ttl = 251  
  AUTHORITY RECORDS:  
  ADDITIONAL RECORDS:  
------------  
Non-authoritative answer:  
Name:	www.google.com  
Address: 142.250.217.100  
  
person1@computer out % nslookup -debug www.google.com 1.1.1.1  
Server:		1.1.1.1  
Address:	1.1.1.1#53  
  
------------  
  QUESTIONS:  
	www.google.com, type = A, class = IN  
  ANSWERS:  
  -> www.google.com  
	internet address = 212.115.105.72  
	ttl = 60  
  AUTHORITY RECORDS:  
  ADDITIONAL RECORDS:  
------------  
Non-authoritative answer:  
Name:	www.google.com  
Address: 212.115.105.72  
  
person1@computer out % nslookup -debug www.google.com 1.0.0.1  
Server:		1.0.0.1  
Address:	1.0.0.1#53  
  
------------  
  QUESTIONS:  
	www.google.com, type = A, class = IN  
  ANSWERS:  
  -> www.google.com  
	internet address = 212.115.105.72  
	ttl = 60  
  AUTHORITY RECORDS:  
  ADDITIONAL RECORDS:  
------------  
Non-authoritative answer:  
Name:	www.google.com  
Address: 212.115.105.72  
  
person1@computer out % nslookup -debug www.google.com 8.8.8.8  
Server:		8.8.8.8  
Address:	8.8.8.8#53  
  
------------  
  QUESTIONS:  
	www.google.com, type = A, class = IN  
  ANSWERS:  
  -> www.google.com  
	internet address = 212.115.105.72  
	ttl = 60  
  AUTHORITY RECORDS:  
  ADDITIONAL RECORDS:  
------------  
Non-authoritative answer:  
Name:	www.google.com  
Address: 212.115.105.72  
```  
  
---  
  
**person1**  
probably umbrella started working.  
  
---  
  
**person3**  
If you are using a 3rd party VPN for “security”, you are…not.  
  
---  
  
**person1**  
That's a very general statement. Perhaps I didn't choose the best VPN.  
Maybe a better statement is, if your configuration doesn't use DNS over TLS in one form or another, you are... not.  
  
---  
  
**person3**  
Without getting too deep into the network and service of it all…when you use the “default”, your machine uses the configured DNS IP address. With Umbrella running, the default DNS IP is configured to be your local network host (127.0.0.1), which happens to be the Umbrella service. When you specify an alternate DNS in your lookup…then your client makes a direct network call to the specified IP address.That part has nothing to do with your VPN.Some VPN clients implement as low level interceptors in the networking stack. That can cause problems with the Umbrella service, which also has a low level process in the networking stack…for monitoring.  
  
Until the DNS lookup is packetized and put into the network stack…your “DNS” configuration should have no interaction with your VPN…unless the VPN client is doing bad things and trying to manipulate your DNS configuration.  
  
There is  _no_  3rd party VPN which improves the security of your device over simply using TLS (HTTPS). VPN only provides security value when the far end is controlled by “yourself”. E.G. company VPN…because we control the server side. The only value of a 3rd party VPN is for anonymization of your client side location…maybe, or getting around regionalization.  
  
---  
  
**person1**  
> There is  _no_  3rd party VPN which improves the security of your device over simply using TLS (HTTPS)  
  
Agree. Unfortunately, there are still legacy things that don't come TLS-wrapped by default. One of them being "classic" DNS protocol.  
  
> That part has nothing to do with your VPN.    
> unless the VPN client is doing bad things and trying to manipulate your DNS configuration.  
  
or the VPN server.  
  
Anyway, if you guys don't see clear signs that I'm breached, I will definitely work on improving the structure of my home network. I don't like ExpressVpn anymore.  
  
---  
  
**person3**  
If you use the default (Umbrella) the DNS call never leaves your local system without TLS protection.  
  
Nothing you have presented indicates a breach. What you are showing is expected and normal behavior.  
  
---  
  
**person1**  
> if you use the default (Umbrella) the DNS call never leaves your local system without TLS protection.  
  
That's what I'm talking about, this 127.0.0.1 DNS acted as if it was also spoofed, until an hour ago or so.  
  
See my messages in this thread today from    
`12:49:47 AM`  (that's slightly past midnight) and  `9:36:53 AM`    
Both times I ran the same command  `nslookup -debug` `[www.google.com](http://www.google.com/)`, but it started behaving differently about an hour ago, showing the correct IP address. That's why I said something must be wrong with this "Umbrella".  
  
BTW other DNSs (direct requests) still act as if they're spoofed.  
  
---  
  
**person4**  
Then stop using ExpressVPN?  
  
---  
  
**person1**  
> Then stop using ExpressVPN?  
  
Yep.  
  
JW how do you explain the behavior of your Umbrella.  
DNS 127.0.0.1 is that umbrella, right?  
  
---  
  
**person4**  
Not necessarily if you're running a second piece of software that also wants to control DNS. We had the same issue with Pritunl when Umbrella was first deployed; both wanted to control 127.0.0.1 DNS. We changed the behavior of Pritunl to stop trying to tweak DNS settings to stop the conflict. Otherwise, you got whatever DNS provider happened to get configured last.  
  
---  
  
**person1**  
I'm not running any software on my Mac, it's just routed through a custom VPN router, which in my network is a standalone device.  
  
---  
  
**person3**  
And, DNS will not (necessarily) return a consistent result for  [google.com](http://google.com/)  even from a single DNS server.  
  
---  
  
**person1**  
It can if somebody's spoofing it.  
  
---  
  
**person4**  
You could also check when you have your VPN turned on if umbrella still looks like this:  
  
---  
  
**person1**  
does it keep history of events?  
I'm a bit surprised, that it was returning spoofed addresses.  
And now, when we started talking about it, it stopped. that's peculiar.  
  
---  
  
**person4**  
We do have logs. Taking a peek  
Umbrella shows no queries in our tenant that returned that address in the last 24 hours.  
  
---  
  
**person1**  
That's scary  
  
---  
  
**person3**  
Umbrella will not indicate results that were sent direct (not to localhost) by a client.  
  
---  
  
**person4**  
You can have multiple DNS listeners listening even on a local port as well.    
That IP does look like a phishing domain. It's only had a DNS name registered for 2 months.  
  
image.png  
  
---  
  
**person1**  
What I published at  `12:49:47 AM`  shows a request to 127.0.0.1  
  
---  
  
**person4**  
image.png  
  
---  
  
**person4**  
that could be why you see inconsistent results, if you have multiple local servers including Umbrella  
  
---  
  
**person1**  
@person4  I didn't install any local DNS servers. How do I ensure Umbrella is the only one?  
  
---  
  
**person1**  
At this point I'm trying to ensure that wherever that comes from, is my router or VPN client/server on my router, and not my (company) Mac.    
Because I can easily nuke/reinstall router, but it's much harder to reset Mac.  
  
---  
  
**person4**  
Maybe something like netstat -avn |grep "\.53 "  
  
---  
  
**person1**  
```  
person1@computer out % netstat -avn |grep "\.53 "  
tcp4    0   0 127.0.0.1.53      *.*          LISTEN    131072 131072 95543   0 00080 00000002 0000000000fe32cf 00000000 00000900   1   0 000001  
udp4    0   0 127.0.0.1.53      *.*                 786896  9216 95543   0 00080 00000004 0000000000fe32ce 00000000 00000800   1   0 000003  
```  
  
---  
  
**person4**  
  
that looks normal  
  
---  
  
**person1**  
Just for my understanding, umbrella sends requests to some secure DNS via TLS?    
what happens if that connection is impossible to establish due to network errors?  
  
Or the endpoint doesn't provide TLS.  
  
I guess what I'm trying to ask is does it fall back to less secure option?  
  
---  
  
**person4**  
Checking to see the answer to that. It does look for access to OpenDNS's servers.  
  
---  
  
**person4**  
You could attempt to query one of these servers and see if it works with your VPN turned on:  [https://docs.umbrella.com/deployment-umbrella/docs/point-your-dns-to-cisco]  
  
Configure DNS to direct traffic from your network to the Cisco Umbrella global network. When a request to resolve a hostname on the internet is made from a network pointed at our DNS addresses, Umbrella applies the security settings in line with your  [policy.To](http://policy.to/)  use Umbrella, you need to explicitly po...  
  
---  
  
**person4**  
BTW, I had IT check and it seems your machine had also fallen out of management. You should be getting a new configuration push happening in the background.  
  
---  
  
**person1**  
> You could attempt to query one of these servers  
  
I'm pretty sure it works now, because Umbrella returns the correct responses ATM.    
But somehow it was giving me different address earlier, so I'm trying to see how it could be done via network traffic control only, without planting malware.  
  
one more piece in the puzzle (Google Chrome):  
  
Screenshot.png  
  
---  
  
**person4**  
Had the management bit been working on your machine, I could have told you exactly what processes were involved when you got the wrong response. For now, you'll need to wait until the config push happens. It should migrate your machine to the new MDM software 'Mosyle'.  
  
---  
  
**person1**  
Sounds great. Thank you guys.  
  
---  
  
**person1**  
It just happened again, but only once:  
```  
person1@computer out % nslookup -debug www.google.com  
Server:		127.0.0.1  
Address:	127.0.0.1#53  
  
------------  
  QUESTIONS:  
	www.google.com, type = A, class = IN  
  ANSWERS:  
  -> www.google.com  
	internet address = 212.115.105.72  
	ttl = 60  
  AUTHORITY RECORDS:  
  ADDITIONAL RECORDS:  
------------  
Non-authoritative answer:  
Name:	www.google.com  
Address: 212.115.105.72  
  
person1@computer out % nslookup -debug www.google.com  
Server:		127.0.0.1  
Address:	127.0.0.1#53  
  
------------  
  QUESTIONS:  
	www.google.com, type = A, class = IN  
  ANSWERS:  
  -> www.google.com  
	internet address = 142.251.211.228  
	ttl = 300  
  AUTHORITY RECORDS:  
  ADDITIONAL RECORDS:  
------------  
Non-authoritative answer:  
Name:	www.google.com  
Address: 142.251.211.228  
```  
  
---  
  
**person1**  
`netstat`  output at that time    
```  
person1@computer out % netstat -avn |grep "\.53 "     
tcp4    0   0 127.0.0.1.53      *.*          LISTEN    131072 131072 95543   0 00080 00000002 0000000000fe32cf 00000000 00000900   1   0 000001  
udp4    0   0 127.0.0.1.35025    127.0.0.1.53            786896  9216  8207   0 00102 00000000 0000000000ffbc7c 00000001 00600800   1   0 000003  
udp4    0   0 127.0.0.1.18115    127.0.0.1.53            786896  9216  8207   0 00102 00000000 0000000000ffbc7b 00000001 00600800   1   0 000003  
udp4    0   0 127.0.0.1.53      *.*                 786896  9216 95543   0 00080 00000004 0000000000fe32ce 00000000 00000800   1   0 000003  
```