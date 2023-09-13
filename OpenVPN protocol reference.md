# Good luck with this shit  
  
# Skip it for now  
  
https://github.com/rusticata/openvpn-parser - parser in Rust  
  
Closest thing to specifications:  
  
-   [https://openvpn.net/index.php/open-source/documentation/security-overview.html](https://openvpn.net/index.php/open-source/documentation/security-overview.html)  
- N/A^  
-   [http://ipseclab.eit.lth.se/tiki-index.php?page=6.+OpenVPN](http://ipseclab.eit.lth.se/tiki-index.php?page=6.+OpenVPN)  
-   OpenVPN source code  
-   OpenVPN wireshark parser  
  
---  
  
https://gerrithub.io/plugins/gitiles/HashBang173/android_external_openvpn/+/refs/heads/cm-9.1.0/ssl.h  
  
source: http://ipseclab.eit.lth.se/tiki-index.php?page=6.+OpenVPN  
  
---  
#### 6.2.1 SSL/TLS riding on UDP  
  
Generally speaking, VPN is just providing a secure data tunnel, the reliable transport layer isn’t a favorite choice as in the IPSec case because of the poor performance and the potential worst situation thanks to the multi-layer time-out and retransmission mechanism needed by TCP.    
When using the SSL/TLS protocol suite to set up a VPN, the first challenge is to switch the native required reliable transport layer of SSL/TLS protocol to the connectionless UDP beneath SSL/TLS. Just like the tricks in DTLS, OpenVPN used a similar technique to get through, which defines one control channel, a equivalent reliable transport layer needed by initial parameters exchange and identity authentication and another data channel, a connectionless UDP-based tunnel to carry VPN data. While IPSec uses two sessions respectively for IKE negotiation and data communication, OpenVPN simply has the control and data channels inside one UDP-based session.    
**Control channel**    
The Control channel acts both in the initial phase tasked with SSL/TLS handshake, including session starting, key exchange, cipher suite agreement, identity authentication, etc. and in the final phase of the connection, where some cleanups are necessary.    
Because of the required connection-oriented transport layer of SSL/TLS handshake protocol, the Control channel itself must compensate for the disadvantage of the unreliable UDP transport layer by simply introducing packet acknowledgement, time-out and retransmission mechanism.. There are two message types in the OpenVPN Control channel, the P_CONTROL message type is a way to implement a TLS cipher packet needed to be encapsulated inside a reliability layer, and the reliability layer is implemented as a straightforward ACK and retransmit model implemented through the P_ACK message type.    
**Data channel**    
The Data channel, established after the SSL/TLS handshake, is a connectionless UDP tunnel where there is no retransmission and ACK mechanism. The P_DATA message type represents encrypted, encapsulated tunnel packets that tend to be either IP packets or Ethernet frames which are essentially the "payload" of the VPN.  
  
#### 6.2.2 TCP/UDP packet format  
  
The following texts extracted from the ssl.h header file in the OpenVPN source code is the constructed packet formats to help understand the practice about SSL/TLS riding on UDP.  
  
OpenVPN can run either on TCP or UDP.  
1. Packet length (16 bits, unsigned)  
   TCP only, always sent as plaintext. Since TCP is a stream protocol, the packet length words  
   define the packetization of the stream.  
2. Packet opcode/Key_id (8bits)  
   TLS only, not used in pre-shared secret mode.  
   P_* constant (high 5 bits)  
   Packet message type.  
   Key_id (low 3 bits)  
   The key_id refers to an already negotiated TLS session. OpenVPN seamlessly renegotiates  
   the TLS session by using a new key_id for the new session. Overlap (controlled by user  
   definable parameters) between old and new TLS sessions is allowed, providing a seamless  
   transition during tunnel operation.  
3. Payload (n bytes)  
   which may be a P_CONTROL, P_ACK, or P_DATA.  
  
  
**Message types**:  
  
1. P_CONTROL_HARD_RESET_CLIENT_V1  
   Key method 1, initial key from client, forget previous state.  
2. P_CONTROL_HARD_RESET_SERVER_V1  
   Key method 2, initial key from server, forget previous state.  
3. P_CONTROL_SOFT_RESET_V1  
   New key, with a graceful transition from old to new key in the sense that a transition window  
   exists where both the old or new key_id can be used. OpenVPN uses two different forms of key_id.  
   The first form is 64 bits and is used for all P_CONTROL messages. P_DATA messages on the other  
   hand use a shortened key_id of 3 bits for efficiency reasons since the vast majority of OpenVPN  
   packets in an active tunnel will be P_DATA messages. The 64 bit form is referred to as a session_id,  
   while the 3 bit form is referred to as a key_id.  
4. P_CONTROL_V1  
   Control channel packet (usually TLS ciphertext).  
5. P_ACK_V1  
   Acknowledgement for P_CONTROL packets received.  
6. P_DATA_V1  
   Data channel packet containing actual tunnel data ciphertext.  
7. P_CONTROL_HARD_RESET_CLIENT_V2  
   Key method 2, initial key from client, forget previous state.  
8. P_CONTROL_HARD_RESET_SERVER_V2  
   Key method 2, initial key from server, forget previous state.  
  
  
  
**Payload:**  
  
1. P_CONTROL* and P_ACK Payload  
   The P_CONTROL message type indicates a TLS ciphertext packet which  
   has been Encapsulated inside of a reliability layer. The reliability  
   layer is implemented as a straightforward ACK and retransmit model.  
2. P_DATA payload:  
   represents encrypted, encapsulated tunnel packets which tend to be  
   either IP packets or Ethernet frames. This is essentially the "payload"  
   of the VPN.  
  
  
**Payload Packet format:**  
  
1. P_CONTROL message format:  
   Local session_id  
     random 64 bit value to identify TLS session  
   HMAC signature of entire encapsulation header for integrity check  
     Valid only if --tls-auth is specified (usually 16 or 20 bytes).  
   Packet-id for replay protection  
     4 or 8 bytes, includes sequence number and optional time_t timestamp  
   P_ACK packet_id array length (1 byte)  
   P_ACK packet-id array (if length > 0)  
   P_ACK remote session_id (if length > 0)  
   Message packet-id (4 bytes)  
   TLS payload ciphertext (n bytes, only for P_CONTROL)  
  
  
Once the TLS session has been initialized and authenticated, the TLS channel is used to exchange random key material for the bidirectional cipher and HMAC keys which will be used to secure the actual tunneled data packets. OpenVPN currently implements two key methods. Key method 1 directly derives keys using random bits obtained from the RAND_bytes OpenSSL function. Key method 2 mixes random key material from both sides of the connection using the TLS PRF mixing function. Key method 2 is the preferred method and is the default for OpenVPN 2.0.  
  
TLS plaintext packet content:  
  if key_method == 1  
     Cipher key length in bytes (1 byte).  
     Cipher key (n bytes).  
     HMAC key length in bytes (1 byte).  
     HMAC key (n bytes).  
     Options string (n bytes, null terminated, client/server options string should match).  
  if key_method == 2  
     Literal 0 (4 bytes).  
     key_method type (1 byte).  
     key_source structure (pre_master only defined for client ->server).  
     options_string_length, including null (2 bytes).  
     Options string (n bytes, null terminated, client/server options string must match).  
     [The username/password data below is optional, record can end at this point.]  
     username_string_length, including null (2 bytes).  
     Username string (n bytes, null terminated).  
     password_string_length, including null (2 bytes).  
     Password string (n bytes, null terminated).  
2. P_DATA message content:  
     HMAC of ciphertext IV + ciphertext (if not disabled by --auth none).  
     Ciphertext IV (size is cipher-dependent, if not disabled by --no-iv).  
     Tunnel packet ciphertext.  
  
  
   P_DATA plaintext:  
     Packet_id (4 or 8 bytes, if not disabled by --no-replay).  
       a. In SSL/TLS mode, 4 bytes are used because the implementation can force a TLS  
          renegotation before 2^32 packets are sent.  
       b. In pre-shared key mode, 8 bytes are used (sequence number and time_t value) to  
          allow long-term key usage without packet_id collisions.  
     User plaintext (n bytes).  
  
  
**Notes:**  
3. ACK messages can be encoded in either the dedicated P_ACK record or they can be prepended to a P_CONTROL message.  
4. P_DATA and P_CONTROL/P_ACK use independent packet-id sequences because P_DATA is an unreliable channel while P_CONTROL/P_ACK is a reliable channel. Each use their own independent HMAC keys.  
5. Note that when --tls-auth is used, all message types are protected with an HMAC signature, even the initial packets of the TLS handshake. This makes it easy for OpenVPN to throw away bogus packets quickly, without wasting resources on attempting a TLS handshake which will ultimately fail.  
  
---