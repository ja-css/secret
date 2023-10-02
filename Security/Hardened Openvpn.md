https://blog.securityevaluators.com/hardening-openvpn-in-2020-1672c3c4135a

# How to Harden OpenVPN in 2020

## This guide will help you configure and secure OpenVPN using the latest best practices.

Since everyone is working from home for the foreseeable future, corporate IT departments are scrambling to bolster existing VPN solutions or deploy new ones as fast as possible. One of the most popular VPN solutions is  [OpenVPN](https://community.openvpn.net/openvpn), either used directly, or through appliances like the commercial  [OpenVPN Access Server](https://openvpn.net/vpn-software-packages/) or third-party VPN gateway products. Some third-party products are not quite upfront about being OpenVPN wrappers, so if you use an SSL VPN Gateway appliance, make sure to double-check the documentation to see if this guide applies to you. If it does, we’ll help you clean up your VPN configuration mess!

let’s clean up this mess

# Why Hardening OpenVPN is Necessary

Most OpenVPN configurations lean heavily on the OpenVPN defaults, which are designed to be widely compatible rather than maximally secure. This is the opposite of what you want on a corporate VPN; since you’re in control of both ends of every connection, you can much more tightly control the clients and can therefore choose  [options](https://openvpn.net/community-resources/reference-manual-for-openvpn-2-4/)  that maximize security. OpenVPN has a pretty staggering amount of them, some of which are deprecated or have subtle security impacts that are not well explained. On top of that, OpenVPN is a pretty old project so there is a lot of advice hanging around on the Internet that is either out of date, incomplete, or just plain wrong. This article aims to be a one stop, up-to-date hardening and configuration guide for OpenVPN in 2020.

# How to Use This Guide

This article will cover a number of hardening options and general best practices broken down into related sections. Each section will include some background, an explanation of the rationale for the specific options it recommends, and a sample configuration snippet that implements it, culminating in a full sample configuration file at the end of the article. If you’re using the OpenVPN community edition (the version that’s available in Linux package managers and on the website), you can copy and paste the directives (customizing as necessary) and build your configuration that way. If you’re using an appliance, consult the manual and the configuration interface to try and find equivalent options and configuration settings. Reading the background and rationale portions of each section can help you find the options in case they’re not named exactly the same way as in the community OpenVPN edition.

At the end of the guide, you’ll have an OpenVPN configuration that uses all modern best practices (known at the time of this guide’s publication) while remaining compatible with all common platforms. There will be a follow-up article after this that gives some points of improvement for extra security depending on your organization’s needs, such as using physical tokens for authentication.

# Basic OpenVPN Configuration

OpenVPN operates using a client/server model with the same configuration system used for both. Configuration parameters are passed either through the command line or, more commonly, through a profile file, a plain text file with the  `.ovpn`  or  `.cfg`  extension. Configuration directives are given one per line, with arguments (if any) for each separated by spaces. Double quotes are used for strings, and lines that begin with  `#`  or  `;`  are comments; the OpenVPN manual recommends that  `#`  be used for text comments and  `;`  be used to comment out directives, but the two characters are otherwise interchangeable.

Some options that accept file paths as an argument, such as the client certificate, can be embedded inside the configuration file. This is advantageous since it reduces the number of files you have to manage. To embed them, you can use an XML-like tag such as this:

<cert>  
(contents of PEM certificate go here)  
</cert>

# Prerequisites

The primary prerequisite for OpenVPN is a  [public-key infrastructure](https://smallstep.com/blog/everything-pki/)  (PKI). In short, you’ll need a certificate authority and the ability to distribute certificates and keys to servers and clients securely. If you’re running a Windows Active Directory domain, you already have a certificate authority that you can use for this, so talk to your AD admin and see how they can help you. If not, you can use  [Step](https://smallstep.com/certificates/),  [XCA](https://hohnstaedt.de/xca/), or the OpenVPN  [easy-rsa](https://github.com/OpenVPN/easy-rsa)  scripts. Be very careful with the CA data, especially if you begin to use it for other purposes than just your OpenVPN setup. The root CA in particular should be stored offline and/or on a hardware security module to prevent theft.

Key distribution can also be a challenge. Since keys can be embedded in OpenVPN configuration files, one option is to email each user their config file in an encrypted zip file and transmit the password to them in another manner such as SMS, but overall this is something that will have to be decided for each individual environment depending on existing infrastructure and policies. Additionally, if you are running OpenVPN via a wrapper such as the official Access Gateway or a third-party SSL VPN gateway, it may have its own way of distributing this information (e.g., via a web page hosted on the device), but that is out of scope of this article.

# Barebones Configuration

If you’re using a VPN appliance, you’re likely not going to be writing these configuration files yourself, but understanding them is helpful to know for debugging purposes.

Here is the most basic OpenVPN server configuration file:
```
port 1194 #listen on port 1194 (default)  
proto udp #use UDP  
dev tun #use a TUN device (layer 3 VPN)  
ca ca.crt #CA certificate(s) in PEM format  
cert server.crt #server certificate chain in PEM format  
key server.key #private key in PEM format  
dh dh2048.pem11 #2048-bit Diffie-Hellman parameters  
server 10.8.0.0 255.255.255.0 #use 10.8.0.0/24 for clients
```
Here is the most basic OpenVPN client configuration file:
```
client #client mode (as opposed to server)  
dev tun #use a TUN device (layer 3 VPN)  
proto udp #use UDP   
remote my-server-1 1194 #the server FQDN or IP and port  
ca ca.crt #CA certificate(s) in PEM format  
cert client.crt #client certificate to connect with in PEM format  
key client.key #private key in PEM format
```
These are taken from the  [OpenVPN sample](https://github.com/OpenVPN/openvpn/tree/master/sample/sample-config-files)  configuration files and are missing a number of desirable options (security and non-security related), so check out the sample file or keep reading for more info.

# Hardening Steps

## General Options

We’ll start with some general best-practice options and easy hardening steps.

First, you’ll need to decide if you want to use UDP or TCP. Using UDP produces higher throughput and lower latency (as it avoids the  [TCP Meltdown Problem](https://openvpn.net/faq/what-is-tcp-meltdown/)) but may not work very well on restrictive networks such as coffee shops. Using TCP, particularly over a commonly-used port such as 443 (HTTPS), is much more likely to work on arbitrary networks. Our example will use UDP on the default port (1194), but swapping to, e.g., TCP 443 has no security implications, so feel free. If you do use TCP mode, there is a configuration option to disable TCP packet coalescing, which you should use as it’s unhelpful for VPNs.

Next, you’ll have to consider whether you want to use TUN or TAP mode. TUN mode is a  [Layer 3](https://en.wikipedia.org/wiki/OSI_model#Layer_3:_Network_Layer)  connection while TAP is a  [Layer 2](https://en.wikipedia.org/wiki/OSI_model#Layer_2:_Data_Link_Layer)  one. In general, use TUN mode — it provides better performance and is the only supported mode on mobile platforms — unless you explicitly need a Layer 2 link such as for carrying Layer 2 broadcast traffic or non-IP protocols for instance.

On both server and client:
```
dev tun # use a TUN device for a layer 3 VPN  
proto udp #TCP is OK but UDP is better  
port 1194 # this is the default, you can use any  
; socket-flags TCP_NODELAY #if using TCP, uncomment this to reduce latency
```
Some other useful options include: 1) a keepalive directive that periodically sends a ping through the tunnel to ensure that the connection is still live and that any intermediate devices such as firewalls with NAT enabled don’t forget about the connection, 2) a directive to allow clients to roam between IP addresses without dropping their connection (good for e.g., mobile phones switching from Wi-Fi to cellular data), and 3) an option for the server that causes it to reject clients whose options don’t match what it expects. On the server only:
```
float #accept authenticated packets from any IP to allow clients to roam  
keepalive 10 60 #send keepalive pings every 10 seconds, disconnect clients after 60 seconds of no traffic  
opt-verify #reject clients with mismatched settings
```
Finally, if you embed all secret information such as the server’s private key directly in the connection profile file rather than referring to an external one, you can additionally set:
```
user nobody  
group nobody  
persist-key #keep the key in memory, don't reread it from disk  
persist-tun #keep the virtual network device open between restarts
```
This will cause the OpenVPN process to drop all its privileges after starting, which makes it more difficult to attack the rest of the server or escalate privileges if the process is compromised due to successful exploitation of a vulnerability (e.g., an unknown/0day buffer overflow vulnerability). This is unfortunately not feasible to set on clients for a number of reasons, such as lack of support on non-Linux OSs.

You’ll note that one thing we did  _not_  include is compression. Compression over VPN links is of very minimal benefit since most traffic is either already compressed (such as images or video), or incompressible (encrypted data such as HTTPS connections). On the other hand, compression can, in some circumstances, be used as an  [Oracle](https://en.wikipedia.org/wiki/Oracle_attack)  to reveal portions of an encrypted channel like a VPN; this was shown to affect OpenVPN in an attack known as  [VORACLE](https://openvpn.net/security-advisory/the-voracle-attack-vulnerability/). As a result, it’s best to leave it off and allow lower-level protocols to implement compression safely if at all.

## TLS, Ciphers, and Key Exchanges

OpenVPN uses TLS for its control channel; the data channel (where your packets actually go) is multiplexed over the same connection but uses a separate cipher and key negotiated over the control channel.

TLS offers essentially 4 points of configuration:

1.  Protocol version. The latest TLS version is 1.3, which is a design overhaul that includes radically changing how the next 3 configuration points are negotiated and which ones are available.
2.  Key exchange mechanism. This is how the client and server agree on the key they use to encrypt packets to each other.
3.  Cipher. The encryption scheme the two parties use to encrypt data (using the key from above).
4.  MAC (Message Authentication Code). This is how the two parties ensure that nobody can modify packets in-flight without being detected.

In general, you want to use the best possible option for each, taking into account what will be available on your clients. OpenVPN is designed to use either OpenSSL or mbedTLS for cryptography, so the availability of TLS versions, ciphers, and so on are dictated by the version of OpenSSL or mbedTLS in use; in practice, most if not all commonly-used versions use OpenSSL, however. OpenVPN on Linux and some Android clients support TLS 1.3, and the macOS client  [TunnelBlick](https://tunnelblick.net/)  supports it with some options, but crucially the Windows client as of this writing does not support it at all. We will, therefore, set TLS 1.3 as a maximum but not the minimum; we’ll use TLS 1.2 as the minimum as it’s the only other secure choice. On both server and client:
```
tls-version-min 1.2  
tls-version-max 1.3 or-highest # use the highest available version if 1.3 isn't available
```
Because of the changes in TLS 1.3, the other three configuration points (Key exchange, cipher, and MAC) will need to be specified twice, once for TLS 1.3, and once for previous versions. However, you’ll want to use essentially the same options for both. The cipher should be either  [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)  in  [GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode)  mode, or a newer construction known as  [ChaCha20](https://en.wikipedia.org/wiki/Salsa20#ChaCha_variant)-[Poly1305](https://en.wikipedia.org/wiki/Poly1305). Both options are  [Authenticated Encryption](https://en.wikipedia.org/wiki/Authenticated_encryption)  schemes, which provide both message secrecy and message integrity at the same time, and as a result, are faster and more secure than having a separate integrity mechanism. We recommend a 128-bit key length for AES (as opposed to the default 256-bit one). 256-bit AES is about 40 percent slower than 128-bit AES and isn’t considered to be significantly stronger in practice, so you can take the performance benefit. If that bothers you, feel free to swap to 256-bit instead.

For key exchanges, you want to use Ephemeral  [Elliptic Curve Diffie-Hellman](https://en.wikipedia.org/wiki/Elliptic-curve_Diffie%E2%80%93Hellman)  (ECDHE). It’s the fastest and most secure way of doing it, and it’s supported on all up-to-date clients. Most importantly, it provides  [Forward Secrecy](https://en.wikipedia.org/wiki/Forward_secrecy), which means that if a key is compromised (such as because a user lost their device), then the attacker can’t decrypt past traffic. When using ECDHE, clients have to additionally negotiate which elliptic curve to use for key exchange out of a large number of standardized curves chosen by different organizations and with different security properties. A discussion of the relative merits of different curves would require a large amount of background on how elliptic curve cryptography works and is not relevant for this document. Instead, we’ll go with the current NSA recommended curve, which is known as  `P-384`  to NIST and  `secp384r1`  to everyone else.

For the MAC, SHA256 is the best compromise between speed and security.

Putting it all together on both server and client:
```
#data channel cipher  
cipher AES-128-GCM  
ncp-disable #don't negotiate ciphers, we know what we want
# TLS 1.3 encryption settings
tls-ciphersuites TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256
# TLS 1.2 encryption settings
tls-cipher TLS-ECDHE-ECDSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-RSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-ECDSA-WITH-AES-128-GCM-SHA256:TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256  
dh none #disable static Diffie-Hellman parameters since we're using ECDHE  
ecdh-curve secp384r1 # use the NSA's recommended curve  
#this tells OpenVPN which side of the TLS handshake it is  
tls-server #tls-client on the client
```
## Handling Certificate Revocations

OpenVPN uses mutual certificate authentication… which means you have to deal with all the complexities that entails.

First off, that means having a way to revoke compromised certificates, which gets very complicated very fast. OpenVPN allows you to specify a CRL (certificate revocation list) file in the configuration, which will contain a list of revoked certificates; connections using those certificates will be rejected. That’s easy to manage on a server since you presumably have ways to access it independently of the VPN, so if you need to revoke a client certificate, you can just log in and add it to the CRL (or preferably develop an automated process for doing so). Managing a CRL on your clients so that you can revoke server certificates is a lot harder since OpenVPN doesn’t have a mechanism akin to  [OCSP](https://en.wikipedia.org/wiki/Online_Certificate_Status_Protocol), which would allow the clients to check if a given server certificate is still valid independently. This presents a problem: You either have to manually distribute an updated CRL or configuration profile, or build an automatic updating mechanism; the former is too slow and manual to be of use, and the latter 1) won’t work on mobile platforms, and 2) would typically use resources (such as a file synchronization app or network share) that sit behind a VPN.

Depending on the level of security appropriate for your use, you could just leave it there and acknowledge the chance that your server could be compromised and its certificate stolen. In that case, the attacker who stole it could perform a man-in-the-middle attack on your clients, potentially compromising them or stealing sensitive data. If that’s unacceptable, you either have to solve the CRL distribution problem, or use an alternative way of limiting the use of stolen certificates; one such alternative is to use short-lived certificates (with a lifetime of one or two days) and renew them automatically. Doing so would mean that a compromised certificate would have a small window of opportunity for use, and an attacker would need to maintain persistent access to a server (which you would hopefully detect and remove) in order to keep getting valid certificates. This could be implemented using  [ACME](https://en.wikipedia.org/wiki/Automated_Certificate_Management_Environment)  and some shell scripting on OpenVPN Community’s server; on third-party gateways, it may or may not be possible depending on what kind of automation is available. In either case, OpenVPN doesn’t offer a mechanism to reload a certificate without restarting the server and disrupting every client in the process. Considering this, some thought will have to be put into how to do this in each environment. The example configuration in this document will simply not handle revocations at all on clients and use a static CRL file on the server.

On the server only:
```
crl-verify /path/to/crlfile
```
## Other Certificate Minutiae

There’s yet more certificate minutiae to deal with here. First, OpenVPN does not perform verification of certificates beyond checking the certificate is signed by the right CA by default.. For a hardened setup, you need to set it to do 2 additional verifications. First, that the remote certificate is being used for its intended purpose (using the certificates  [Extended Key Usage](https://en.wikipedia.org/wiki/X.509#Extensions_informing_a_specific_usage_of_a_certificate)  flags). Second, that the client is talking to the right server. The first one is supported on both client and server, but the second isn’t easily supported on the server — there isn’t an easy way to check that a user’s certificate matches their username/password, so for the example config we’ll ignore that possibility. In the “Extra Credit” article following this one, we’ll show a skeleton of how to enforce certificate/username matching. For now, use this on the server:
```
remote-cert-tls client #require client certificates to have the correct extended key usage  
verify-client-cert require #reject connections without certificates
```
For the client to validate the server certificate fully, you’ll need its common name (which should match its intended hostname).
```
remote-cert-tls server  
verify-x509-name your-server-common-name.example.com name
```
On both of them, set this additional option to require certificates to use modern key sizes and signing algorithms:
```
tls-cert-profile preferred
```
## User Authentication

Once a user has established a TLS connection with the server, there is an optional additional authentication step where the user can be asked for a username and password for the server to validate before allowing them to complete the VPN connection. This is highly recommended as it results in multi-factor authentication; the certificate in the profile is “something you have”, and the user’s credentials are “something you know”. The list of usernames and passwords is usually outsourced to another authentication database such as LDAP. This is accomplished using either a binary plugin system or a custom script that is called by OpenVPN for each login attempt, and because of its varied nature we will consider it out of scope for this document. Instead, we will use  [PAM](https://en.wikipedia.org/wiki/Pluggable_authentication_module)  to authenticate against the server’s local user list as OpenVPN includes a plugin to do just that. On the server:
```
plugin /usr/share/openvpn/plugin/lib/openvpn-auth-pam.so login
```
On the client, we need to instruct OpenVPN to ask the user for credentials. By default, it will allow the user to save their password to disk, which depending on the platform may not use secure key storage and defeats the purpose of using users’ passwords as MFA since both the key and the password become ‘something you have’ (a file stored on a disk). While your users may not appreciate it very much, you can disable the option to do so. On the client’s configuration for maximum security:
```
auth-nocache #don't cache credentials in memory  
setenv ALLOW_PASSWORD_SAVE 0 #disallow saving of passwords
```
Since the connection is renegotiated periodically (either on a timer, or because of network interruptions), this will result in the user being forced to re-enter their password quite often — which leads to a very bad user experience. To combat this, you can instruct the server to send clients a longer-term authentication token that they can use when renegotiating the connection. In other words, during the initial connection the user will enter their password as normal, but future connections (as long as the user doesn’t manually disconnect the VPN) will use the token instead for as long as it’s valid. This can be set up on the server with the following directive:
```
auth-gen-token 43200 #lifetime of token in seconds; this is 12 hours
```
On the client, you also have to  **remove**  the  `auth-nocache`  directive since it will prevent the VPN from caching the token.

The net result of this configuration is that users will be prompted for credentials when they first connect, and every 12 hours thereafter, but the actual password they use will never be cached on disk or in memory.

# Putting it all together

If you’re using OpenVPN community, use this server configuration file as a template:
```
server  
proto udp  
port 1194  
dev tun # layer 3 vpn  
<ca>  
(CA certificate in PEM form)  
</ca>  
<cert>  
(server certificate in PEM form)  
</cert>  
<key>  
(private key contents in PEM form)  
</key>  
dh none #don't use key exchanges with static parameters  
server 10.8.0.0 255.255.255.0 # use 10.8.0.0/24 for clients  
float # allow clients to roam  
keepalive 10 60 #send keepalive pings every 10 seconds, disconnect clients after 60 seconds of no traffic  
opt-verify #reject clients with mismatched settings  
user nobody  
group nobody  
persist-key #keep the key in memory, don't reread it from disk  
persist-tun #keep the virtual network device open between restarts  
tls-version-min 1.2  
tls-version-max 1.3 or-highest # use the highest available version if 1.3 isn't available  
#data channel cipher  
cipher AES-128-GCM  
ncp-disable #don't negotiate ciphers, we know what we want
# TLS 1.3 encryption settings
tls-ciphersuites TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256
# TLS 1.2 encryption settings
tls-cipher TLS-ECDHE-ECDSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-RSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-ECDSA-WITH-AES-128-GCM-SHA256:TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256  
dh none #disable static Diffie-Hellman parameters since we're using ECDHE  
ecdh-curve secp384r1 # use the NSA-'s recommended curve  
tls-server #be the server side of the TLS handshake  
crl-verify /path/to/crlfile  
remote-cert-tls client #require client certificates to have the correct extended key usage  
verify-client-cert require #reject connections without certificates  
tls-cert-profile preferred #require certificates to use modern key sizes and signatures  
plugin /usr/share/openvpn/plugin/lib/openvpn-auth-pam.so login # use PAM for login  
auth-gen-token 43200 #lifetime of token in seconds; this is 12 hours
```
For clients, use this:
```
client  
proto udp  
port 1194  
remote your-domain-name.example.com  
dev tun  
<ca>  
(CA certificate in PEM form)  
</ca>  
<cert>  
(client certificate in PEM form)  
</cert>  
<key>  
(private key contents in PEM form)  
</key>  
tls-version-min 1.2  
tls-version-max 1.3 or-highest # use the highest available version if 1.3 isn't available  
#data channel cipher  
cipher AES-128-GCM  
ncp-disable #don't negotiate ciphers, we know what we want
# TLS 1.3 encryption settings
tls-ciphersuites TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256
# TLS 1.2 encryption settings
tls-cipher TLS-ECDHE-ECDSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-RSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-ECDSA-WITH-AES-128-GCM-SHA256:TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256  
ecdh-curve secp384r1 # use the NSA-'s recommended curve  
tls-client #be the client side of the TLS handshake  
tls-cert-profile preferred #require certificates to use modern key sizes and signatures  
remote-cert-tls server #require server certificates to have the correct extended key usage  
verify-x509-name your-domain-name.example.com name  
auth-nocache #don't cache credentials in memory  
setenv ALLOW_PASSWORD_SAVE 0 #disallow saving of passwords
```
Reminder: security best practices are always changing, so some of the recommended options might change in the future. Feel free to give a shout if it needs to be updated!
