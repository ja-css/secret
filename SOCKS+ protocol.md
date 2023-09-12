Protocol is based on SOCKS5 over TLS.


The handshake is exactly the same as SOCKS5, the differences start in how multiple connections over the same channel are supported.

after a handshake is done and a channel is established, we start multiplexing client channels over this connection

exchange:

1) SocksPortUnificationServerHandler

final byte versionVal = in.getByte(readerIndex);

case SOCKS5:
	logKnownVersion(ctx, version);
	p.addAfter(ctx.name(), null, socks5encoder);
	p.addAfter(ctx.name(), null, new Socks5InitialRequestDecoder());
	break;

1 - client is sending Socks5InitialRequest
1 - server decodes DefaultSocks5InitialRequest via Socks5InitialRequestDecoder
1 - server responds with DefaultSocks5InitialResponse in SocksPlusServerHandler, adds Socks5InitialRequestDecoder to pipeline

//we assume no authentication
```
2 - client sends Socks5CommandRequest
2 - server decodes DefaultSocks5InitialRequest with Socks5CommandRequestDecoder
2 - server connects to the endpoint, responds with DefaultSocks5CommandResponse, sets relay/pump
```
-=-=-=-=-=-=-=-=-=-

Socks5ServerEncoder
```
if (msg instanceof Socks5InitialResponse) {
    encodeAuthMethodResponse((Socks5InitialResponse) msg, out);
} else if (msg instanceof Socks5PasswordAuthResponse) {
    encodePasswordAuthResponse((Socks5PasswordAuthResponse) msg, out);
} else if (msg instanceof Socks5CommandResponse) {
    encodeCommandResponse((Socks5CommandResponse) msg, out);
} else {
    throw new EncoderException("unsupported message type: " + StringUtil.simpleClassName(msg));
}
```

```
private static void encodeAuthMethodResponse(Socks5InitialResponse msg, ByteBuf out) {  
  out.writeByte(msg.version().byteValue());  
  out.writeByte(msg.authMethod().byteValue());  
}
```

```  
private static void encodePasswordAuthResponse(Socks5PasswordAuthResponse msg, ByteBuf out) {  
  out.writeByte(0x01);  
  out.writeByte(msg.status().byteValue());  
}  
```

```  
private void encodeCommandResponse(Socks5CommandResponse msg, ByteBuf out) throws Exception {  
  out.writeByte(msg.version().byteValue());  
  out.writeByte(msg.status().byteValue());  
  out.writeByte(0x00);  
  
 final Socks5AddressType bndAddrType = msg.bndAddrType();  
  out.writeByte(bndAddrType.byteValue());  
  addressEncoder.encodeAddress(bndAddrType, msg.bndAddr(), out);  
  
  out.writeShort(msg.bndPort());  
}
```