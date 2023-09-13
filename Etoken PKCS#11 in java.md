TODO: Trying to gather info on this one  
  
https://stackoverflow.com/questions/23319281/java-keytool-keystore-shows-no-entries-for-etoken  
  
  
https://docs.oracle.com/javase/9/security/pkcs11-reference-guide1.htm#JSSEC-GUID-05DE5039-C62A-42B9-8F8E-DB6DB1B0CD44  
  
  
  
  
So far this succeeded on Java8:  
`keytool -keystore NONE -storetype PKCS11 -providerClass sun.security.pkcs11.SunPKCS11 -providerArg config -list`  
