- Get public key from a certificate file
  `openssl x509 -pubkey -noout -in client-certificate.pem`

- Get public key from a client certificate (e.g. in SSL)
  `Base64.getEncoder().encodeToString(sslSession.getPeerCertificates()[0].getPublicKey().getEncoded())`

- Save certificate to a file
```
final FileOutputStream os = new FileOutputStream("cert.cer");
os.write("-----BEGIN CERTIFICATE-----\n".getBytes("US-ASCII"));
os.write(Base64.encodeBase64(cert.getEncoded(), true));
os.write("-----END CERTIFICATE-----\n".getBytes("US-ASCII"));
os.close();
```

- encrypt with RSA test
  https://www.devglan.com/online-tools/rsa-encryption-decryption

RSA by itself is deterministic. However, to get a better security and prevent attackers from guessing the encrypted information, the encryption is done not on the pure "data" but on "data"+"some-random-pattern" ([wikipedia](http://en.wikipedia.org/wiki/RSA_%28algorithm%29#Padding_schemes))

```
Cipher cipher = Cipher.getInstance("RSA");  
cipher.init(Cipher.ENCRYPT_MODE, sslSession.getPeerCertificates()[0].getPublicKey());  
String encrypt = Base64.getEncoder().encodeToString(cipher.doFinal("hello".getBytes()));
```