#### Do we need a lightweight proxy server? In Rust?
- TLS OVER TLS chaining u Ha java 3ae6eLLIbCR nucaTb, a Tbl roBopuLLIb Rust.
* Ho Ha cepBepe HeT 4euHuHra, ToJIbKo Ha KJIueHTe!
- TeM He MeHee, ynpaBJIeHue CepTuqpuKaTaMu y4uTb 3aHoBo Ha Rust Towe He Auc.
- Tak ewe u PKCS#11, KoTopbIu HuKTo He CannopTuT!

---

The plan

1. The process of creating certificates, client and server.
   Kind of described in the doc: **"EToken, certificates, ssh"**
   *TODO: run on minilaptop*

2. ProxAK Vert.x TLS - client and server certificate, secure comm

3. ProxAK Vert.x SOCKS+ (from SOCKS 5)

4. ProxAK Vert.x SOCKS+ proxy chain app + UI

---

Non-essential

5. Support for SOCKS4 and SOCKS5 in chain app


6. Support for UDP via `UDP ASSOCIATE`


7. Vert.x TLS with PKCS#11


*TODO: this is probably going to be a tough one. Also try yubikey as PKCS#11*
> YubiKey devices can be used with various cryptographic libraries and middleware on Linux, including OpenSC, which provides PKCS#11 support. Yubico also provides their own command-line tools and libraries, such as yubico-piv-tool and libu2f-host, which allow for interaction with YubiKey devices on Linux.

---

Krakozyabra found:

AND WTF IS THIS?
Fri Jun 9 2023 7:34 PM PT
19:30:49.486 [vert.x-eventloop-thread-0] ERROR com.microsocks.SimplePumpClient - Failed to connect to 2606:4700:0:0:0:0:6812:15e2:80
io.netty.channel.AbstractChannel$AnnotatedNoRouteToHostException: No route to host: /[2606:4700:0:0:0:0:6812:15e2]:80
Caused by: java.net.NoRouteToHostException: No route to host
