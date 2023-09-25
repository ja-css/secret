### Disclaimer:
> Unfortunately, SafeNet E-Token modules are not available for arm64.  
For that reason a dedicated and isolated x86 or amd64 station might be required to support both CA and write tokens without copying private keys around.
We will need such station to import tokens anyway, so why not run CA on it?

> Update: We won't be copying private keys around, if we use the technique described in [EToken: Generate keys on token and run CSR.md](EToken%3A%20Generate%20keys%20on%20token%20and%20run%20CSR.md)  
The idea is to generate private key right on the token, so it doesn't get exposed ever at all anywhere or copied. 
Obviously, this approach has its drawbacks, because the key will be lost with token and can't be recovered.  
Thus, it's not advisable to use for cryptocurrency account keys. And even in other applications you need to design your routines around this fact, with key deprecation and rotation built-in, and/or backup keys.  
But other than that, it's the most secure way to store keys.

# Install modules

Once LUKS is installed on Raspberry Pi, next steps are:

The following modules should be installed:  
- `easy-rsa`
- `wipe`


1. EasyRSA can be found in router packages.
'easyrsa' is needed to issue certificates. More info in [EToken, certificates, ssh.md](EToken%2C%20certificates%2C%20ssh.md)
2. `wipe` is installed as a standalone module.  
`wipe` is needed to incinerate sensitive files, notably private keys.
