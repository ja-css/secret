https://blog.securityevaluators.com/how-to-harden-openvpn-in-2020-part-2-da51acab3ea8

# How to Harden OpenVPN in 2020 — Part 2

### This guide will help you configure and secure OpenVPN using the latest best practices.

blog.securityevaluators.com

In the previous article [Hardened Openvpn.md](Hardened%20Openvpn.md), I laid out a framework for building a modern, hardened OpenVPN server/client configuration. At the end, I noted there were some additional hardening steps that would be nice to take for extra security. In particular:

1.  Using an additional static TLS key in the initial TLS handshake to prevent denial-of-service attacks.
2.  Storing keys in hardware cryptographic devices to prevent exfiltration.
3.  Using multi-factor authentication with time-based one time passwords (TOTP, AKA Google Authenticator)
4.  Closing the small security hole created because OpenVPN doesn't by default check that client certificates match client usernames.
5.  Instructing OpenVPN to apply additional exploit mitigation measures to itself after initialization.

Like the previous article, this will be slightly complicated by the fact that many installations of OpenVPN don't use the community edition server directly, but wrap it in some other interface or appliance. It will likely be impossible to apply the latter two hardening steps in that case, and may also be impossible to perform the first or third depending on what options are exposed. If you use a wrapper or appliance and the security benefits of these additional configuration steps seem like something you want and the appliance doesn't offer the options to do so, check with the support team for the product and see if they're applying them already or if they can expose the relevant options in their interface.

From now on, I will assume you have a working OpenVPN configuration (probably one vaguely like the one I developed in the previous article) and that you have a working knowledge of OpenVPN syntax and general Linux/Unix administration basics. If you haven't read the previous article, you might want to check it out to make sure you haven't missed any of the hardening steps there as well.

# Caveats

In general, these steps are relegated to an "extra credit" article rather than being part of the main one for a combination of reasons:

1.  Not all of them will be applicable for every user. If you're using a third party gateway, it may not have the option in question, or it may be doing something different internally. Similarly, it may not work at all if you're not running the right OS or lack the requisite hardware.
2.  Some of them require substantial environment-specific customization.
3.  Some may not be unilateral improvements depending on your specific environment and needs.

With those caveats in mind, on to the actual configuration!

# Using a Static TLS Key in OpenVPN Handshakes

As mentioned in the previous article, OpenVPN uses TLS for its control channel. Configured and used correctly, TLS is secure against pretty much all (known) attacks. However, there are a handful of minor flaws to consider:

1.  Because authentication occurs  _after_  the initial handshake (which requires a small but nonzero amount of computation), an attacker could open thousands of connections at once and consume a large amount of resources including CPU and network.
2.  If you use UDP mode (which you should if possible since it's much more performant), there's also a risk of being used as a vector for  [UDP reflection](https://blog.cloudflare.com/reflections-on-reflections/)  attacks since the initial UDP TLS packets will not require authentication.
3.  The OpenVPN protocol is easily identifiable on network sniffers, firewall logs, and so on. This may be undesirable if you want your VPN usage to be secret from governments, ISPs, etc.
4.  TLS in its current form is not  [quantum-resistant](https://en.wikipedia.org/wiki/Post-quantum_cryptography); the elliptic curve Diffie-Hellman key exchange (ECDHE) used in modern TLS relies on the difficulty of certain problems in elliptic curves, which are easily solvable by an advanced quantum computer. As far as anybody knows, no such computer exists currently (nor is one likely to become available in the near future), but if one is eventually produced it could be used to decrypt previously recorded traffic flows. Depending on your threat model, this likely isn't that concerning, but it's worth pointing out.

The first flaw is an issue on web services as well, but they typically just implement connection rate limiting to mitigate it. The other two are OpenVPN-specific, since regular HTTPS services don't use UDP and there isn't usually a concern about being identified as using HTTPS.

So how do we resolve these flaws? OpenVPN includes the  `tls-crypt`  option¹, which encrypts and authenticates the entire TLS channel (including the initial packets) with a static, pre-shared symmetric key. Any packets that are not correctly encrypted and authenticated are simply dropped. As long as adversaries don't possess the key, they cannot:

1.  Open new connections (resolving the DoS and UDP reflection issues)
2.  Decode the otherwise-unencrypted handshake (solving the identification issue)
3.  See the underlying ECDHE and use a quantum computer (now or later) to break the ECDHE and decrypt the VPN traffic.

This is not a perfect solution—since the key has to be shared between every party, any client that is compromised or that is malicious can still perform all four of the attacks identified above. If a key  _was_  compromised, you would have to rotate the PSK by issuing new configuration files to each client. However, at that point the security of the protocol only degrades to where it would be if the option wasn't used at all, so in such a case it may not be urgently necessary depending on your threat model.

[**1**] This replaces (and is mutually incompatible with) the older  `tis-auth`  option, which only prevented the first two attacks since it simply added an HMAC authentication tag to each TLS packet.

# Using the  `tls-crypt`  Option

Using  `tls-crypt`  is easy. First, generate an appropriate key by issuing the following command:

openvpn --genkey --secret tls-crypt.key

The key will be written to  `tls-crypt.key`. After that, embed it in both your server and client configuration files like so:

# …other configuration directives above
<tls-crypt>  
# contents of tls-crypt.key go here  
</tls-crypt>

And that’s it! All TLS communication will now be encrypted and authenticated as described above.

# Storing Keys in Hardware

Up until now, the hardening steps I’ve shown were mostly oriented at  _preventing_  breaches. However, it’s important to realize that all the hardening in the world won’t prevent 100 percent of breaches — if nothing else, users get phished or have their devices stolen all the time — so it’s almost equally important to consider what happens when a breach occurs. If a user’s device is compromised, the attacker will almost certainly be able to exfiltrate their key (since it’s just stored in a file), and would probably be able to keylog their username and password. With those two things, the attacker can then access the VPN as that user.

There are two was to make achieving that goal more difficult for attackers. The first is to use a one-time password such as via the time-based one time password (TOTP) algorithm, which is discussed below. The other is to store the key not in the configuration profile, but in somewhere that won’t allow it to be exfiltrated — a hardware secure enclave. Modern Windows PCs include a  [Trusted Platform Module (TPM)](https://en.wikipedia.org/wiki/Trusted_Platform_Module). Newer Macs include the  [T2 chip](https://support.apple.com/en-us/HT208862), which has similar functionality, and all modern smartphones include roughly equivalent hardware as well. You can also use  [Yubikeys](https://www.yubico.com/)  or smartcards to store user keys if you want to issue portable tokens rather than keys bound to physical devices. Both cases are applications of Multi-Factor Authentication, combining “something you know” (a password) with “something you have” (a physical device).

For our purposes, all of the above perform similar functions: they generate, store, and use cryptographic secrets without ever exposing them to the underlying OS. For example, an attacker that compromises a PC will be able to ask the TPM to perform operations such as sign challenges, but will not be able to actually steal the keys; they will have to maintain continuous access to the compromised PC in order to e.g., ask the TPM to sign TLS handshake packets. This is much noisier, dependent on the compromised user actually being online, and is much less convenient for the attacker than simply stealing the key, so it’s a massive improvement.

OpenVPN on Windows supports using the Windows Crypto API to perform cryptographic operations, which means that any certificate available to the system’s certificate manager can be used to connect to a VPN endpoint; this includes keys stored on the TPM, or on any smart cards or cryptographic dongles like Yubikeys connected to the system. On Windows and other desktop platforms, OpenVPN additionally supports loading external  [PKCS#11](https://en.wikipedia.org/wiki/PKCS_11)-compatible modules to perform crypto operations. On Linux, there is a  [library](https://github.com/tpm2-software/tpm2-pkcs11)  for using the TPM this way; one could hypothetically be built for macOS and its T2 chip, but as far as I am aware no such module exists as of this writing. As for smartcards, many (but not all!) are supported by the  [OpenSC](https://github.com/OpenSC/OpenSC)  project, which includes a PKCS#11 module. It also supports many cryptographic dongles such as Yubikeys.

# Certificate Provisioning

Before using a certificate for OpenVPN (or anything else), you of course have to issue it. The process for generating them on a cryptographic dongle is different than normal certificates, since you never actually end up with a file containing the private key.

For Windows machines, generating a key and issuing a certificate on the TPM can either be done using Active Directory Certificate Services (ADCS) or manually. To issue it using ADCS, create a template for a TLS client certificate and specify the Microsoft Platform Crypto Provider as the cryptographic provider, then create and issue a certificate as normal. To do so without ADCS (either because you are not using Active Directory or because your OpenVPN CA isn’t your ADCS CA), see  [my other blog post about this topic](https://polansky.co/blog/tpm-backed-certificates-windows/).

For all platforms, using a PKCS#11 smartcard or Yubikey requires installing OpenSC (or another PKCS#11 driver if it doesn’t support your hardware) and OpenSSL. The OpenSC PKCS#11 module  [readme](https://github.com/OpenSC/libp11#using-p11tool-and-openssl-from-the-command-line)  has a walkthrough of how to use to issue OpenSSL commands that interact with PKCS#11 devices.

# Using Microsoft CryptoAPI Certificates in OpenVPN

Once you have issued a certificate and the Microsoft Crypto API is aware of it (either on the TPM, on a smartcard compatible with the default smartcard driver, or on a smartcard with its own CryptoAPI driver you’ve installed), you can specify it to OpenVPN using its  `cryptoapicert`  option. The option takes a single argument, a "select string" that tells OpenVPN how to find the certificate; the  [manual](https://openvpn.net/community-resources/reference-manual-for-openvpn-2-4/)  describes how to construct this string:

> _To select a certificate, based on a substring search in the certificate’s subject:_
>
> `_cryptoapicert "SUBJ:Peter Runestig"_`
>
> _To select a certificate, based on certificate’s thumbprint:_
>
> `_cryptoapicert "THUMB:f6 49 24 41 01 b4 …"_`
>
> _The thumbprint hex string can easily be copy-and-pasted from the Windows Certificate Store GUI._

# Using PKCS#11 Certificates in OpenVPN

While cross-platform, this is unfortunately more complicated.

First, locate the appropriate PKCS#11 module. For OpenSC, you’ll want to locate the  `opensc-pkcs11.so`  file, which might be in  `/usr/lib`  depending on where your distro puts it. On macOS, the  `brew`  installation of OpenSC puts it at  `/usr/local/lib/onepin-opensc-pkcs11.so`. On Windows, it goes into the OpenSC installation directory. Use the appropriate value as the argument for the option  `pkcs11-providers provider`.

Second, identify the ID of the certificate you want to use. It will vary by device and by PKCS#11 provider, so you can use  `openvpn --pkcs11-providers /path/to/provider.so --show-pkcs11-ids`  to list the ones on the system. Then add the option  `pkcs11-id your_id`  to the configuration file. There are additional options that you may need or want to configure depending on your use case and hardware such as pin caching and what specifically the OpenVPN program requests from the crypto device, so check the manual out (search for  `pkcs11`  to see the relevant options) and try it them yourself to see what works.

# Hardware-Backed Cryptography on Mobile Devices

As of this writing, both the community-supported  [OpenVPN for Android](https://play.google.com/store/apps/details?id=de.blinkt.openvpn&hl=en_US)  app and the official  [OpenVPN Connect](https://play.google.com/store/apps/details?id=net.openvpn.openvpn&hl=en_US)  apps support using the Android Key Store (which is hardware backed on all recent devices) for certificate storage. Simply import a profile with no certificate or private key and the apps will allow you to pick a certificate from the system store to use. The official iOS  [OpenVPN Connect](https://play.google.com/store/apps/details?id=net.openvpn.openvpn&hl=en_US)  app works similarly using the iOS certificate store. You can also issue mobile profiles containing SCEP (Simple Certificate Enrollment Profile) information bundled with OpenVPN profiles to iOS devices, which makes VPN provisioning a single step process.

# What About the OpenVPN Server?

Key compromise on an OpenVPN server would be even worse than on a client, since the attacker would be able to man-in-the-middle all traffic going to or from the server. If you followed the guide in the previous article, they would at least not be able to decrypt past traffic, nor decrypt any future traffic they were not actively intercepting thanks to the forward secrecy offered by ECDHE, but even being able to mount an active man-in-the-middle is bad enough to warrant trying to prevent key compromise. The solution is the same as on the clients — store the key in a cryptographic device. This is a little tricker on the server side, since most crypto devices are relatively slow. Commercial Hardware Security Modules (HSMs) are usually capable of performing operations at high speeds (especially for RSA at smaller key sizes and for elliptic curve cryptography), so it may be worth using one. They typically have PKCS#11 interfaces, so you can follow the same steps for clients to use them.

# Time-Based One Time Passwords

If managing hardware certificates is too much work, another option is to use OpenVPN’s challenge plugin functionality, which allows you to specify a library that will issue and validate challenges in addition to users’ passwords. Users will be prompted for the challenge when the connection is initiated, and possibly when it is renegotiated depending on your settings. One such module is  [openvpn-totp](https://github.com/evgeny-gridasov/openvpn-otp), which prompts for and validates  [Time-Based One Time Passwords](https://github.com/evgeny-gridasov/openvpn-otp#using-openvpn-otp-for-multi-factor-authentication), which are generated based on a shared secret between the server and client. Users will use a smartphone app such as Google Authenticator (available on  [Android](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en_US)  and  [iOS](https://apps.apple.com/us/app/google-authenticator/id388497605)) to generate short numeric codes which they will enter when prompted.

When used as in the challenge/response mode (a guide for which is in the  [README](https://github.com/evgeny-gridasov/openvpn-otp#using-openvpn-otp-for-multi-factor-authentication)), this module provides multi-factor authentication: “Something you know” (a password) is combined with “something you have” (a phone). With this configuration, an attacker would have to both steal a user’s credentials and gain access to their phone in order to access the VPN.

It is also possible to handle this at the directory layer, which is useful if you use a gateway without plugin support. In that case, configure OpenVPN to use your directory, and consult your directory’s manual for how to enable it. For instance, OpenLDAP has a  [TOTP overlay module](https://github.com/openldap/openldap/tree/master/contrib/slapd-modules/passwd/totp). Once you have done so, users will have to login by inputting both their TOTP code and their password in the same password field in order to login.

# Binding Certificates to Users

When a user connects to the OpenVPN server, it checks certain things about the user:

1.  Is their certificate valid? That is, are the signatures in the chain valid, was it issued by a trusted CA, does it have the correct Extended Key Usage flags, etc.
2.  Is their username and password valid according to whatever authentication mechanism is configured?
3.  If you enable strict options checking (recommended, see the previous article), are their options valid?

Note one thing it does not check: Does the presented certificate match the user that’s using it? Without that check, an attacker may use any certificate (either gained legitimately or compromised) to login as any user provided they know the user’s password. To prevent this, you can use the OpenVPN shell script verification option, which causes OpenVPN to run a shell script using environment variables to pass information about the user in order to validate their credentials. The environment variables include information about the certificate (its CN, its serial number, etc.) and the username and password. The following script demonstrates how to check that a user’s certificate matches their username:

```
#!/bin/bash# credit to this ServerFault user: [https://serverfault.com/a/360399](https://serverfault.com/a/360399)
# username and common_name must be the same to allow access.
# users are not allowed to share their cert
if [ $username != $common_name ]; then  
echo "$(date +%Y%m%d-%H%M%S) DENIED  username=$username cert=$common_name" >> /var/log/openvpn-access.log  
exit 1  
fi# supply your own validate_username_and_password() function
# e.g., check against LDAP using ldapbind
if ! validate_username_and_password(); then  
echo "$(date +%Y%m%d-%H%M%S) DENIED  username=$username cert=$common_name" >> /var/log/openvpn-access.log  
exit 1  
fiecho "$(date +%Y%m%d-%H%M%S) GRANTED username=$username cert=$common_name" >> /var/log/openvpn-access.logexit 0
```

Save the script on the server somewhere, then customize the  `validate_username_and_password()`  function to match your environment (e.g., use the command line LDAP tools to check against LDAP, or use a Python script to check against the local PAM subsystem). Then add the following directives to the server:
```
script-security 2 # allow external scripts  
auth-user-pass-verify /path/to/script.sh via-env
```
# Additional Exploit Mitigations

The suggested configuration in the previous article included instructing OpenVPN to drop its privileges to the  `nobody`  user/group after initialization. This way, if a security issue in the server was exploited and granted an attacker the ability to execute code or disclose files as the OpenVPN process, the impact of the exploit would be limited. The attacker would only be able to read files that  `nobody`  could read and wouldn't be able to interact with the wider system in a meaningful way without a separate escalation-of-privilege vulnerability. However, there's still a reasonably wide attack surface for an attacker to work with in this scenario: They can make network connections, interact with the kernel using any unprivileged system call, and read any world-readable file. In an ideal world, none of these things would be overly concerning since the server wouldn't have any important data left with weak permissions, the internal network the VPN grants access to would be patched and have firewalls in place, the kernel would have no escalation-of-privilege issues, and so on. We however live in a world where none of those things are guaranteed, so we must adopt a defense-in-depth strategy and apply exploit mitigations so as to limit the impact of any security issues.

Linux has a feature known as  [chroot](https://en.wikipedia.org/wiki/Chroot)  (“change root”) that allows a process to tell the kernel it wants to treat a specific directory as if it was the root directory of the system. In other words, all further file system calls (listing directories, opening files, and so on) will be relative to that directory, and it will no longer be able to access any files outside of it. This is commonly used for bootstrapping a system or rescuing a broken one (for instance,  `chroot`-ing onto a system with a broken kernel from a live disk to reinstall it), but it's also a useful security feature. Applications can  `chroot`themselves into a 'jail' that contains only the files they need, and then if their process is later exploited by an attacker, the attacker will not be able to read or access any other files.

For the case of OpenVPN, it’s very easy to identify which files need to be in its  `chroot`: none, except any scripts your specific configuration uses. Everything else is only required during initialization (assuming you set the options to persist relevant data in memory), so it isn't necessary to have it available after initialization, when the OpenVPN daemon will  `chroot`  itself.
```
persist-key  
persist-tun  
chroot /path/to/dir
```
# SELinux

SELinux (Security Enhanced Linux) is a set of Linux security modules that allow fine-grained access controls to be applied to users and processes. While full configuration of SELinux is out of scope of this article, the  [CentOS wiki](https://wiki.centos.org/HowTos/SELinux) has an excellent introduction, including example policies. Once you have SELinux configured for your system, you can use the reference OpenVPN policy (in the SELinux Reference Policies repository as a base. You apply that policy by the standard SELinux configuration mechanisms, customizing it slightly if you need to grant access to additional calls such as those needed by any custom scripts or plugins. However, for even greater security you can also customize it to be even more restrictive and remove access to system calls and directories not necessary after initialization (such as opening additional sockets), then pass the  `setcon <context>`  configuration directive to OpenVPN to instruct it to apply the given policy to itself after initialization. This will of course require customization depending on your environment but will yield the most hardened server possible.

Note that combining SELinux with  `chroot`  is possible, but requires some extra work:

> _You can of course combine [_`_setcon_`_,_ `_user_` _and_ `_chroot_`_], but please note that since_ `_setcon_` _requires access to_ `_/proc_` _you will have to provide it inside the chroot directory (e.g. with_ `_mount --bind_`_)._

# Conclusion

This concludes our two-part series on OpenVPN hardening. At this point, you should have configured OpenVPN to use modern ciphers and key exchanges, considered using physical security tokens for authentication, reduced the impact of certificate and credential theft, balanced user experience with the need to reduce the presence of credentials in memory, and applied exploit mitigations to the server process so as to reduce the impact of any 0day security issues that affect it.
