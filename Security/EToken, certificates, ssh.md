# EToken, certificates, ssh  
  
[0. Install debian](#install-debian)  
[1. Configure EasyRSA and create certificates (debian)](#configure-easyrsa-and-create-certificates-debian)  
[2. Install Safe Net Authentication client](#install-safe-net-authentication-client)  
[3. SSH Cert](#ssh-cert)  
[4. Connecting via SSH to AWS servers](#connecting-via-ssh-to-aws-servers)  
[5. Secure delete everything except for public keys](#secure-delete-everything-except-for-public-keys)  
[6. Create your own ssh server and add public key to ssh, prohibit login/password entry](#create-your-own-ssh-server-and-add-public-key-to-ssh-prohibit-loginpassword-entry)  
  
  
  
## 0. Install debian  
  
- Use image without firmware  
- Install GNOME (for SafeNet)  
  
## 1. Configure EasyRSA and create certificates (debian)  
  
Memo: EasyRSA doesn't support root certificate on PKCS#11 token, for that reason we need to maintain our certificate store on a very well encrypted machine. MB: contribute to code?  
Quote:  
https://docs.bigchaindb.com/projects/server/en/v1.3.0/production-deployment-template/client-tls-certificate.html  
  
Install easyrsa on Debian  
`sudo apt install easyrsa`  
OR  
`sudo apt install easy-rsa`  
  
The binary `easyrsa` can be found in  
`/usr/share/easy-rsa`  
  
---  
1.1 Init CA  
`sudo ./easyrsa init-pki`  
`sudo ./easyrsa build-ca nopass`  
  
---  
1.2 **Create a certificate request (CSR):**  
  
`sudo ./easyrsa gen-req NAME nopass`  
This will create  
`/usr/share/easy-rsa/pki/reqs/NAME.req`  
`/usr/share/easy-rsa/pki/private/NAME.key`  
  
ALTERNATIVELY: import an external request:  
`sudo ./easyrsa import-req /path/to/NAME.req NAME`  
  
---  
then sign a CSR creating a cert
`./easyrsa sign-req client NAME`  
  
---  
As a result, a certificate will be created at:  
`/usr/share/easy-rsa/pki/issued/NAME.crt`  
  
Once you have signed it, you can send the signed certificate and the CA certificate back to the requestor.  
(CA certificate is `pki/ca.crt`)  
  
---  
1.3 create pem  
`cat pki/issued/NAME.crt pki/private/NAME.key > NAME.pem`  
  
---  
1.4 create pfx  
`openssl pkcs12 -export -out NAME.pfx -inkey pki/private/NAME.key -in pki/issued/NAME.crt`  
  
---  
1.5 create pub for ssh  
`ssh-keygen -f NAME.pem -y > NAME.pub`  
If this causes error: `Permissions xxxx for 'NAME.pem' are too open`, do `chmod 400 NAME.pem`.

OR
```
openssl x509 -pubkey -noout -in MY.crt > MY.pem
ssh-keygen -f MY.pem -i -m PKCS8 > MY.openssh.pub
```
  
## 2. Install Safe Net Authentication client  

> Important: token passwords in Mac version and Linux version are not fully compatible. I've tested alphanumeric and it works fine, however a symbol like `#` causes issues and passwords won't work on either Linux or MacOS.  

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


### Mac  
https://www.certsign.ro/en/support/safenet-installing-the-device-on-macos/  
Version for BigSur: [here](http://support.certsign.ro/safenet/SafeNetAuthenticationClient.10.2.111.0.dmg)  
  
Quote:  
[SafeNet Authentication Client_9.0_Users_Guide.pdf](https://knowledge.digicert.com/content/dam/digicertknowledgebase/attachment/solution/SO28683/SafeNet%20Authentication%20Client_9.0_Users_Guide.pdf)  
  
Importing a Certificate to a Token  
The following certificate types are supported:  
- *.pfx  
- *.p12  
- *.cer  
  
I.e. ***.pem is NOT supported.**  
  
### Ubuntu 20  
  
Download the client here:  
https://knowledge.digicert.com/generalinformation/INFO1982.html  
Linux    
[https://www.digicert.com/StaticFiles/SAC_10_7_Linux_GA.zip](https://www.digicert.com/StaticFiles/SAC_10_7_Linux_GA.zip)  
  
run  
`sudo apt upgrade`  
`sudo apt update`  
  
run `dpkg -i Installation/Standard/DEB/safenetauthenticationclient_...deb`  
then restart  
  
### Debian  
  
- Same as Ubuntu  
- make sure root's path has  
  `echo "export PATH=$PATH:/sbin:/usr/sbin:/usr/local/sbin" >> ~/.bashrc`  
- make sure easy-rsa is installed (step 1), it has some dependencies needed for the client to work.  
- Use GNOME  
- restart after install  
  
## 3. SSH Cert  
  
Quote:  
https://security.stackexchange.com/questions/32768/converting-keys-between-openssl-and-openssh  
SSH uses its own public key format.  
  
If you just want to share the private key, the OpenSSL key generated by your example command is stored in `private.pem`, and it should already be in PEM format compatible with (recent) OpenSSH. To extract an OpenSSH compatible public key from it, you can just run the following command:  
  
3.1 Create ssh public key out of pem file  
```  
ssh-keygen -f NAME.pem -y > NAME.pub  
```  
  
## 4. Connecting via SSH to AWS servers  
  
Importing a public ssh key is done by creating a key pair on AWS side (via browser or client).  
In browser: - `Network & Security` -> `Key Pairs` -> `Actions` -> `Import key pair`  
Client help here:  
https://docs.aws.amazon.com/cli/latest/reference/ec2/import-key-pair.html  
  
  
### Route 1: simple ssh-keygen (UNUSED, just for the reference)  
  
`ssh-keygen -t rsa -C "my-key" -f my-key`  
This will create `my-key` and `my-key.pub`.  
  
  
* to connect to Amazon Linux on AWS - use user **ec2-user**:  
  `ssh -i my-key ec2-user@HOSTNAME`  
  
* to connect to Ubuntu Linux on AWS - use user **admin**:  
  `ssh -i my-key admin@HOSTNAME`  
  
  
### Route 2: easyrsa + pem certificate  
  
- Create certificates (1.1), create *pem (1.2)  
- Create and import public key to create a key pair on AWS side (see 3.1).  
- login like this:  
  `ssh -i NAME.pem ec2-user@HOSTNAME`  
  
  
### Route 3: and the most important one - SSH with Safe Net token  
Install a good SafeNet Authentication client (2).  
  
#### Important: token passwords in Mac version and Linux version are not fully compatible. I've tested alphanumeric and it works fine, however a symbol like `#` causes issues and passwords won't work on either Linux or MacOS.  
  
Install a pfx certificate from (1.3) on SafeNet Token.  
Pretty straightforward via GUI client (import certificate).  
  
  
Use the following command to log in to the same server for **Route 2** with eToken:  
MacOS (tested on BigSur + SAC 10.2):  
`ssh -I /usr/local/lib/libeToken.dylib ec2-user@HOSTNAME`  
Linux (tested on Ubuntu + SAC 10.7):  
`ssh -I /usr/lib/libeToken.so ec2-user@HOSTNAME`  
SFTP (untested):  
`sftp -o "SmartcardDevice=/usr/lib/libeToken.so" user@example.com`  
  
Example successful connection output on MacOS (for the reference):  
```  
john@MacBook-Pro certs % ssh -I /usr/local/lib/libeToken.dylib ec2-user@ec2-3-95-176-151.compute-1.amazonaws.com  
Enter PIN for 'token2':  
Last login: Sun Oct 30 06:01:28 2022 from 173.239.197.56  
    
 __|  __|_  )  
 _|  (     /    Amazon Linux 2 AMI  
___|\___|___|  
    
https://aws.amazon.com/amazon-linux-2/  
13 package(s) needed for security, out of 16 available  
Run "sudo yum update" to apply all updates.  
[ec2-user@ip-172-31-82-206 ~]$  
```  
  
#### Multiple certificates:  
Multiple certificates are supported on the token and can be used for different servers; apparently they are somehow automatically matched automatically by ssh client / pkcs11.  
  
CONFUSING: Please note, that in case there is no matching certificate found, the connection will fail with Permission Denied, and it won't ask a token pin, which might look like the token doesn't work properly.  
```  
john@MacBook-Pro certs % ssh -I /usr/local/lib/libeToken.dylib ec2-user@ec2-35-175-244-208.compute-1.amazonaws.com  
ec2-user@ec2-35-175-244-208.compute-1.amazonaws.com: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).  
```  
  
## 5. Secure delete everything except for public keys  
  
Quote:  
https://www.groovypost.com/howto/securely-delete-files-in-linux/  
  
### secure-delete  
`sudo apt-get install secure-delete`  
`sudo srm delete-this-file.txt`  
  
### wipe  
`sudo apt-get install wipe`  
`wipe /home/lori/Documents/delete-this-file.txt`  
  
  
## 6. Create your own ssh server and add public key to ssh, prohibit login/password entry  
  
Quote:  
https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server  
  
### Copying an SSH Public Key to Your Server  
  
The content of your  `id_rsa.pub`  file will have to be added to a file at  `~/.ssh/authorized_keys`  on your remote machine somehow.  
  
Make sure the `~/.ssh` directory is created:  
`mkdir -p ~/.ssh`  
  
Create or modify the `authorized_keys` file:  
`echo PUBLIC_KEY_STR >> ~/.ssh/authorized_keys`  
  
In the above command, substitute the `PUBLIC_KEY_STR` with the output from the `cat ~/.ssh/id_rsa.pub` command that you executed on your local system. It should start with `ssh-rsa AAAA...` or similar.  
  
### Quick method  
  
`cat id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"`  
  
### Disabling Password Authentication on your Server  
  
- `sudo nano /etc/ssh/sshd_config`  
- update contents: `PasswordAuthentication no`  
- `sudo systemctl restart ssh`  
  
### Log In  
`ssh -I /usr/lib/libeToken.so admin@34.203.194.238`  
`ssh -I /usr/local/lib/libeToken.dylib IP_ADDR`