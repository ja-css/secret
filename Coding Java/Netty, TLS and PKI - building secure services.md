# Netty, TLS and PKI - building secure services  
# TODO: write this article one day  
  
Fuck safenet shit only good for signatures
try this
https://developers.yubico.com/YubiHSM2/Component_Reference/PKCS_11/


> Prereq:  
> See doc [EToken, certificates, ssh.md](..%2FSecurity%2FEToken%2C%20certificates%2C%20ssh.md) for easyRSA CA routines.  
> Issue certificates using the above info.


- Memo:  
You can view the algorithms used for key encryption in a PKCS#12 certificate using the keytool utility, which is part of the Java Development Kit (JDK). Here's how you can do it:  
`keytool -list -v -storetype PKCS12 -keystore your_certificate.pfx`  
This, however, doesn't show PBKDF algo.


## 1. Encrypt and decrypt strings with PKCS#12 / PFX file-based certificate

```
import javax.crypto.Cipher;
import java.io.FileInputStream;
import java.security.KeyStore;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.cert.X509Certificate;
import java.util.Collections;
import java.util.Base64;

public class LoadPKCS12Certificate {
    public static void main(String[] args) throws Exception {
        // Path to your PKCS#12 certificate file
        String keystorePath = "/Users/john/certs/rpi2.pfx"; // Replace with your PKCS#12 file path
        char[] keystorePassword = "qwerty".toCharArray(); // Replace with your keystore password

        // Create a KeyStore instance for PKCS#12
        KeyStore keyStore = KeyStore.getInstance("PKCS12");

        // Load the PKCS#12 certificate file
        try (FileInputStream fis = new FileInputStream(keystorePath)) {
            keyStore.load(fis, keystorePassword);
        }

        // List the aliases in the keystore (in this case, your certificate alias)
        for (String alias : Collections.list(keyStore.aliases())) {
            System.out.println("Alias: " + alias);

            // Get the X.509 certificate for the alias
            X509Certificate certificate = (X509Certificate) keyStore.getCertificate(alias);

            // Print certificate details (you can do more with the certificate if needed)
            System.out.println("Subject: " + certificate.getSubjectDN());
            System.out.println("Issuer: " + certificate.getIssuerDN());
            System.out.println("Serial Number: " + certificate.getSerialNumber());
            System.out.println("Valid From: " + certificate.getNotBefore());
            System.out.println("Valid Until: " + certificate.getNotAfter());

            PublicKey publicKey = certificate.getPublicKey();
            String publicKeyAlgorithm = publicKey.getAlgorithm();
            String signatureAlgorithm = certificate.getSigAlgName();
            int keySize = publicKey.getEncoded().length * 8;

            System.out.println("Public Key Algorithm: " + publicKeyAlgorithm);
            System.out.println("Signature Algorithm: " + signatureAlgorithm);
            System.out.println("Key Size: " + keySize + " bits");

            //--------------------------------

            // The string to be encrypted
            String originalString = "Hello, this is a secret message.";

            // Encrypt the string using the public key
            String encryptedString = encryptWithPublicKey(originalString, publicKey);
            System.out.println("Encrypted: " + encryptedString);

            // Decrypt the string using the private key
            PrivateKey privateKey = (PrivateKey) keyStore.getKey(alias, keystorePassword);
            String decryptedString = decryptWithPrivateKey(encryptedString, privateKey);
            System.out.println("Decrypted: " + decryptedString);
        }
    }

    // Encrypt a string using the recipient's public key
    private static String encryptWithPublicKey(String input, PublicKey publicKey) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);

        byte[] encryptedBytes = cipher.doFinal(input.getBytes("UTF-8"));
        return Base64.getEncoder().encodeToString(encryptedBytes);
    }

    // Decrypt a string using the private key
    private static String decryptWithPrivateKey(String encryptedInput, PrivateKey privateKey) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);

        byte[] encryptedBytes = Base64.getDecoder().decode(encryptedInput);
        byte[] decryptedBytes = cipher.doFinal(encryptedBytes);
        return new String(decryptedBytes, "UTF-8");
    }
}
```

Tpe6oBaHuR:  
1. accept only self-issued certificates, client and server  
2. support for PKCS#11 certificates, client and server  




