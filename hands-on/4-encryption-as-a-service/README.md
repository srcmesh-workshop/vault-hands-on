# Encryption as a Service: Transit Secrets Engine

## Enable Transit Secrets Engine

```bash
$ vault secrets enable [-path=<MOUNT_PATH>] transit
```

## Create A Encryption Key

```bash
$ vault write -f <MOUNT_PATH>/keys/<ENCYPTION_KEY_NAME>
```

`-f`: Allow the operation to continue with no key=value pairs. This allows
writing to keys that do not need or expect data.

## Encrypt secrets

```bash
$ vault write <MOUNT_PATH>/encrypt/<ENCYPTION_KEY_NAME> \
    plaintext=<BASE64_STRING>
    
$ vault write user-0-transit/encrypt/first_key \
    plaintext=$(base64 <<< "4111 1111 1111 1111")

Key            Value
---            -----
ciphertext     vault:v1:UF1l758xIryu02RlOILhTCmbhA+lRT+hI3w7KofXsT3QEBX8NGjpfZzvf/waCg93
key_version    1
```

## Decrypt ciphertext

```bash
$ vault write <MOUNT_PATH>/decrypt/<ENCYPTION_KEY_NAME> ciphertext=<CIPHERTEXT>
    
$ vault write user-0-transit/decrypt/first_key ciphertext=vault:v1:UF1l758xIryu02RlOILhTCmbhA+lRT+hI3w7KofXsT3QEBX8NGjpfZzvf/waCg93

Key          Value
---          -----
plaintext    NDExMSAxMTExIDExMTEgMTExMQo=
```

Check decryption result

```bash
$ echo -n "NDExMSAxMTExIDExMTEgMTExMQo=" |base64 -D
4111 1111 1111 1111
```

## Rotate the encryption key 

```bash
$ vault write -f <MOUNT_PATH>/keys/<ENCYPTION_KEY_NAME>/rotate

$ vault write -f user-0-transit/keys/first_key/rotate
Success! Data written to: user-0-transit/keys/first_key/rotate
```

Encrypt again

```bash
vault write user-0-transit/encrypt/first_key \
    plaintext=$(base64 <<< "4111 1111 1111 1111")
Key            Value
---            -----
ciphertext     vault:v2:XqNG9Trs8gjdmhQors5Zg6BkOPzBH5KfPR9aOdbsQYT8OwwEXpV2o6VgIwDcJU24
key_version    2
```

You are still able to decrypt ciphertext with old version encryption key.

```bash
$ vault write user-0-transit/decrypt/first_key ciphertext=vault:v1:UF1l758xIryu02RlOILhTCmbhA+lRT+hI3w7KofXsT3QEBX8NGjpfZzvf/waCg93
Key          Value
---          -----
plaintext    NDExMSAxMTExIDExMTEgMTExMQo=
```

## Automatic key rotation

```bash
$ vault write <MOUNT_PATH>/keys/<ENCYPTION_KEY_NAME>/config auto_rotate_period=<TIME>

$ vault write user-0-transit/keys/first_key/config auto_rotate_period=24h
Success! Data written to: user-0-transit/keys/first_key/config
```

Check `auto_rotate_period` is non `0` vaule.

```bash
$ vault read user-0-transit/keys/first_key
Key                       Value
---                       -----
allow_plaintext_backup    false
auto_rotate_period        24h
deletion_allowed          false
derived                   false
exportable                false
keys                      map[1:1649979331 2:1649979664]
latest_version            2
min_available_version     0
min_decryption_version    1
min_encryption_version    0
name                      first_key
supports_decryption       true
supports_derivation       true
supports_encryption       true
supports_signing          false
type                      aes256-gcm96
```

## Specify Minimum encryption key version

Set `min_decryption_version` to `2`.

```bash
$ vault write user-0-transit/keys/first_key/config min_decryption_version=2
Success! Data written to: user-0-transit/keys/first_key/config
```

The older ciphertext is now longer able to decrypt.

```bash
$ vault write user-0-transit/decrypt/first_key ciphertext=vault:v1:UF1l758xIryu02RlOILhTCmbhA+lRT+hI3w7KofXsT3QEBX8NGjpfZzvf/waCg93
Error writing data to user-0-transit/decrypt/first_key: Error making API request.

URL: PUT http://35.236.166.63:8200/v1/user-0-transit/decrypt/first_key
Code: 400. Errors:

* ciphertext or signature version is disallowed by policy (too old)
```

Set `min_decryption_version` to `1`.

```bash
$ vault write user-0-transit/keys/first_key/config min_decryption_version=1
Success! Data written to: user-0-transit/keys/first_key/config
```

```bash
$ vault write user-0-transit/decrypt/first_key ciphertext=vault:v1:UF1l758xIryu02RlOILhTCmbhA+lRT+hI3w7KofXsT3QEBX8NGjpfZzvf/waCg93
Key          Value
---          -----
plaintext    NDExMSAxMTExIDExMTEgMTExMQo=
```