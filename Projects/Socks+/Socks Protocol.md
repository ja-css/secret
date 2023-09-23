Vert.x doesn't support byte-based ip addresses directly, but it can parse them from strings.  
e.g:  
https://github.com/eclipse-vertx/vert.x/blob/master/src/test/java/io/vertx/test/proxy/Socks4Proxy.java#L76  
```  
String ip = getByte4(buffer.getBuffer(4, 8));  
...  
private String getByte4(Buffer buffer) {  
    return String.format("%d.%d.%d.%d", buffer.getByte(0), buffer.getByte(1), buffer.getByte(2), buffer.getByte(3));  
}  
```  
  
Main flow:  
	1. Read SOCKS version  
	2. Process request  
		- read request  
		- open stream to destination address/port  
		- send `success` to the client  
		- pump incoming and outgoing data  
	3. Close socket  
  
  
## SOCKS 4  
  
### Request  
  
| field name      | size            |  
|-----------------|-----------------|  
| version         | byte            |  
| command         | byte            |  
| port data       | byte[2]         |  
| ipv4addressdata | byte[4]         |  
| username        | null-terminated |  
| hostname        | null-terminated |  
  
#### Commands:  
    SOCKS_COMMAND_CONNECT = 1;  
    SOCKS_STATUS_FAILURE = 0x5b;  
    SOCKS_STATUS_SUCCESS = 0x5a;  
  
#### Hostname:  
    only present if `ipv4addressdata` is 0.0.0.0  
  
#### Response:  
	final byte[] response = new byte[8];    
	response[0] = 0;    
	response[1] = (byte) code;  
  
## SOCKS 5  
  
### Request - authentication  
  
| field name      | size            |  
|-----------------|-----------------|  
| version         | byte            |  
| nmethods        | byte            |  
| methods         | byte[nmethods]  |  
  
#### Auth methods:  
    SOCKS5_AUTH_NONE = 0;  
    FAILURE = 0xFF;  
  
#### Response - authentication:  
	final byte[] response = new byte[2];    
	response[0] = SOCKS5_VERSION;    
	response[1] = (byte) method;  
	  
---  
  
### Request - connection  
  
| field name      | size            |  
|-----------------|-----------------|  
| version         | byte            |  
| command         | byte            |  
| reserved        | byte            |  
| addressType     | byte            |  
| addressBytes    | `*`             |  
| portBytes       | byte[2]         |  
  
#### AddressTypes  
	SOCKS5_ADDRESS_IPV4 = 1;  
	SOCKS5_ADDRESS_IPV6 = 4;  
	SOCKS5_ADDRESS_HOSTNAME = 3;  
   
#### AddressBytes `*`  
	for SOCKS5_ADDRESS_IPV4 - byte[4]  
	for SOCKS5_ADDRESS_IPV6 - byte[16]  
	  
	for SOCKS5_ADDRESS_HOSTNAME -   
	| field name      | size            |  
	|-----------------|-----------------|  
	| length          | byte            |  
	| addressData     | byte[length]    |  
  
#### Commands:  
	SOCKS5_COMMAND_CONNECT = 1;  
  
#### Statuses  
	SOCKS5_STATUS_SUCCESS = 0  
	SOCKS5_STATUS_FAILURE = 1  
	SOCKS5_STATUS_CONNECTION_REFUSED = 5  
	SOCKS5_STATUS_COMMAND_NOT_SUPPORTED = 7  
  
#### Response - connection:  
| field name      | size            |  
|-----------------|-----------------|  
| SOCKS5_VERSION  | byte            |  
| status          | byte            |  
| reserved (0)    | byte            |  
| addressType     | byte            |  
| addressBytes    | `*`             |  
| portBytes       | byte[2]         |  
