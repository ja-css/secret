from: https://cloud.google.com/kms/docs/wrapping-a-key

# Wrapping a key using OpenSSL on Linux

This topic shows how to manually wrap a key before  [importing the key](https://cloud.google.com/kms/docs/importing-a-key)  into Cloud KMS. You only need to follow the instructions in this topic if you do not want to use the Google Cloud CLI to automatically wrap the key before importing it. For an overview of the differences, refer to  [How key import works](https://cloud.google.com/kms/docs/key-import#key_import_flow).

## Retrieve the wrapping key

Get the damned wrapping key.

## Set up environment variables

The OpenSSL commands require several file paths as input values. Define environment variables for the file paths to make it easier to run the commands. Make sure you have access to write to the directories you define below.

**Note:** If desired, you can save all of these commands to a shell script and run the script to set the variables all at once. You may need to adjust some of the values for each key you want to wrap.

1.  Set the  `PUB_WRAPPING_KEY`  variable to the full path to the wrapping key you downloaded from the import job. The wrapping key ends in  `.pem`.
```
    PUB_WRAPPING_KEY="WRAPPING_KEY_PATH"
```

2.  Set the  `TARGET_KEY`  variable to the full path to the unwrapped (target) key.
```
    TARGET_KEY=TARGET_KEY_PATH
```
    Replace  `TARGET_KEY_PATH`  with the path to the  `.bin`  file for symmetric keys or the path to the  `.der`  file for asymmetric keys.

3.  If wrapping with RSA-AES, set the  `TEMP_AES_KEY`  variable to the full path to the temporary AES key.
```
    TEMP_AES_KEY=TEMP_AES_KEY_PATH
```

4.  Set the  `WRAPPED_KEY`  variable to the full path where you want to save the wrapped target key that is ready for import.
```
    WRAPPED_KEY=WRAPPED_KEY_PATH
```

5.  Verify that all the environment variables are set correctly using the following commands:
```
    echo "PUB_WRAPPING_KEY: " ${PUB_WRAPPING_KEY}; \
    echo "TARGET_KEY: " ${TARGET_KEY}; \
    echo "TEMP_AES_KEY: " ${TEMP_AES_KEY}; \
    echo "WRAPPED_KEY: " ${WRAPPED_KEY}
```

When the variables are set correctly, you are ready to wrap the key. There are two approaches as described below: with  [RSA only](https://cloud.google.com/kms/docs/wrapping-a-key#rsa_wrap)  or with  [RSA-AES](https://cloud.google.com/kms/docs/wrapping-a-key#rsa_aes_wrap).

## Wrap the key

### Wrap the key with RSA

In this approach, the target key is wrapped in an RSA block. The target key size is therefore limited. For example, you can't use this method to wrap another RSA key. The supported import methods are  `rsa-oaep-3072-sha256`  and  `rsa-oaep-4096-sha256`.

-   Wrap the target key with the wrapping public key using the  `CKM_RSA_PKCS_OAEP`  algorithm:
```
    openssl pkeyutl \
    -encrypt \
    -pubin \
    -inkey ${PUB_WRAPPING_KEY} \
    -in ${TARGET_KEY} \
    -out ${WRAPPED_KEY} \
    -pkeyopt rsa_padding_mode:oaep \
    -pkeyopt rsa_oaep_md:sha256 \
    -pkeyopt rsa_mgf1_md:sha256
```

### Wrap the key with RSA-AES

In this approach, the target key is wrapped with a temporary AES key. The temporary AES key is then wrapped by the RSA key. These two wrapped keys are concatenated and imported. Because the target key is wrapped using AES rather than RSA, this approach can be used to wrap large keys. The supported import methods are  `rsa-oaep-3072-sha1-aes-256`,  `rsa-oaep-4096-sha1-aes-256`,  `rsa-oaep-3072-sha256-aes-256`  and  `rsa-oaep-4096-sha256-aes-256`.

1.  Generate a temporary random AES key that is 32 bytes long, and save it to the location identified by  `${TEMP_AES_KEY}`:
```
    openssl rand -out "${TEMP_AES_KEY}" 32
```

2.  Wrap the temporary AES key with the wrapping public key using the  `CKM_RSA_PKCS_OAEP`  algorithm. If the import method is either  `rsa-oaep-3072-sha1-aes-256`  or  `rsa-oaep-4096-sha1-aes-256`, use  `sha1`  for  `rsa_oaep_md`  and  `rsa_mgf1_md`. Use  `sha256`  for  `rsa-oaep-3072-sha256-aes-256`  and  `rsa-oaep-4096-sha256-aes-256`.
```
    openssl pkeyutl \
    -encrypt \
    -pubin \
    -inkey ${PUB_WRAPPING_KEY} \
    -in ${TEMP_AES_KEY} \
    -out ${WRAPPED_KEY} \
    -pkeyopt rsa_padding_mode:oaep \
    -pkeyopt rsa_oaep_md:{sha1|sha256} \
    -pkeyopt rsa_mgf1_md:{sha1|sha256}
```

3.  Set the  `OpenSSL_V110`  variable to the path of your  `openssl.sh`  script. If you followed the instructions for  [patching and recompiling OpenSSL](https://cloud.google.com/kms/docs/configuring-openssl-for-manual-key-wrapping)  exactly, you can use this command without modifying the value of the variable.
```
    OPENSSL_V110="${HOME}/local/bin/openssl.sh"
```

4.  Wrap the target key with the temporary AES key using the  `CKM_AES_KEY_WRAP_PAD`  algorithm, and append it to the  `WRAPPED_KEY`.
```
    "${OPENSSL_V110}" enc \
    -id-aes256-wrap-pad \
    -iv A65959A6 \
    -K $( hexdump -v -e '/1 "%02x"' < "${TEMP_AES_KEY}" ) \
    -in "${TARGET_KEY}" >> "${WRAPPED_KEY}"
```
The  `-iv A65959A6`  flag sets A65959A6 as the Alternate Initial Value. This is required by the  [RFC 5649](https://tools.ietf.org/html/rfc5649)  specification.


## What's next

-   The wrapped key saved at  `WRAPPED_KEY`  is now ready to be imported.