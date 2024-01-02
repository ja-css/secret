# If the CKA_EXTRACTABLE attribute is FALSE, then the key cannot be wrapped.

p11ls -l /usr/local/lib/libeToken.dylib
p11od -l /usr/local/lib/libeToken.dylib prvk/100

> 
> 
> > Important: token passwords in Mac version and Linux version are not fully compatible. I've tested alphanumeric and it works fine, however a symbol like `#` causes issues and passwords won't work on either Linux or MacOS.

> Important: Installing certificates on the token using `Safe Net Authentication client` results in private key installed with permissions `AT_SIGNATURE`, as opposed to `AT_KEYEXCHANGE`. As a result of that the private key can only be used for signature creation, but not for encrypting and decrypting data.  
There are alternative ways to import the certificate, but each has its drawbacks.
>- The document [EToken, OpenSSL encrypt files.md](EToken%2C%20OpenSSL%20encrypt%20files.md) describes using OpenSC's `pkcs11-tool` to import private key, which results in private key being created with permissions to "sign", "encrypt" and "wrap". Those permissions can't be configured. We can encrypt, but according to [EToken vulns.md](EToken%20vulns.md), we shouldn't allow the same key to wrap and encrypt, see doc for details.
>- The document [EToken: Generate keys on token and run CSR with pkcs11-tools.md](EToken%3A%20Generate%20keys%20on%20token%20and%20run%20CSR%20with%20pkcs11-tools.md) describes a way to generate private key directly on the token using MasterCard's `pkcs11-tools`. It allows to set attributes for the key, so we can allow "sign" and "encrypt" without "wrap", but this key is not exportable and will perish with the token. So, when token breaks you can't recover your key to a different token. This might be desirable in a number of scenarios, but not in all of them.
>- As of now I'm still looking for a way to import a private key with sign and encrypt only. MasterCard's `pkcs11-tools` don't seem to allow importing a private key (more info here: https://github.com/Mastercard/pkcs11-tools/blob/master/docs/MANUAL.md#p11importcert), so one direction worth investigating is minimal code changes in the original OpenSC's `pkcs11-tool` to allow setting private key attributes (here https://github.com/OpenSC/OpenSC/blob/master/src/tools/pkcs11-tool.c#L3927).  
   P.S. Looks like `pkcs11-tool` already has something of the sort here, need to make sure I'm using the latest version on Linux. (https://github.com/OpenSC/OpenSC/blob/master/src/tools/pkcs11-tool.c#L3927):  
   `{ "usage-sign",		0, NULL,		OPT_KEY_USAGE_SIGN },`  
   `{ "usage-decrypt",	0, NULL,		OPT_KEY_USAGE_DECRYPT },`  
   `{ "usage-derive",	0, NULL,		OPT_KEY_USAGE_DERIVE },`  
   `{ "usage-wrap",	0, NULL,		OPT_KEY_USAGE_WRAP },`  
   added on Feb 1, 2020, commit: https://github.com/OpenSC/OpenSC/commit/0cd19b59e19a7129f4064fcb94e0e39615ec0cdc  
   On Linux the options exist, but they don't seem to set attributes as expected. Need to debug and fix the code, potentially compare with `pkcs11-tools`. (here: https://github.com/Mastercard/pkcs11-tools/blob/master/lib/pkcs11_pubk.c#L1022-L1046)  
   Maybe, you shouldn't do import private key, only unwrap?  
   https://www.ibm.com/docs/en/zos/2.5.0?topic=objects-pkcs-11-unwrap-key-csfpuwk-csfpuwk6
   >  * A new secret key object is created with the decrypted key value.
>* How to make keys non-exportable?  
   CKA_EXTRACTABLE Attribute: The CKA_EXTRACTABLE attribute is commonly used to control whether a key can be exported.  
   If you set CKA_EXTRACTABLE to CK_FALSE when creating or importing the key, it should make the key non-exportable.
   This attribute is often set during the key generation process.
>* Investigate wrap/unwrap. How do I make sure one can't wrap and steal a private key?
>* https://cloud.google.com/kms/docs/wrapping-a-key
>* `p11unwrap` syntax  
   you must at least provide:  
   -f, the path to a wrapping key file produced by p11wrap  
   In addition, PKCS#11 attributes can be specified, that will override attributes from the wrapping key file.
>
>
> - Looks to me that this Could be done:
>   - 1. Import key with regular permissions
>   - 2. Wrap
>   - 3. Unwrap with new permissions
> - OR
    >   - 1. Create and wrap key with OpenSSL
>   - 2. Unwrap key with new permissions


# Direction 1
Howj
>   - 1. Import key with regular permissions
>   - 2. Wrap
>   - 3. Unwrap with new permissions




p11wrap -a oaep -i aes-wrapping-key -w rsa-wrapping-key -o aes-wrapping-key.wrap  
p11wrap -w 1 -i 1 -l /usr/local/lib/libeToken.dylib  -o aes-wrapping-key.wrap  


no wrap by default?
https://github.com/OpenSC/OpenSC/issues/1913

p11ls -l /usr/local/lib/libeToken.dylib
p11od -l /usr/local/lib/libeToken.dylib pubk/100



p11wrap -w rsa-wrapping-key -i 1 -l /usr/local/lib/libeToken.dylib  -o asdfghj

pkcs11-tool --unwrap --mechanism RSA-PKCS --id 22 -i aes_wrapped.key --key-type AES: --application-id 90 --applicatin-label unwrapped-key