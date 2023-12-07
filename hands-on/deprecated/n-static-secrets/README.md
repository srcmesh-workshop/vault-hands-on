# Static Secrets

## Enable Key/Value Secrets Engine

```bash
# Path: the path to visit secrets
# kv: Key/Value Secrets Engine
$ vault secrets enable -path="<path>" -description="<description>" kv
```

Here we use `kv-v1` for demonstration.

```bash
$ vault secrets enable -path="kv-v1" -description="Test K/V v1" kv
```

## Store the Google API key

```bash
$ vault kv put kv-v1/<PATH> <KEY>=VALUE>
$ vault kv put kv-v1/eng/apikey/Google key=AAaaBBccDDeeOTXzSMT1234BB_Z8JzG7JkSVxI
$ vault kv get kv-v1/eng/apikey/Google
```

## Store the root certificate for MySQL

```bash
$ openssl req -x509 -sha256 -nodes -newkey rsa:2048 -keyout selfsigned.key -out cert.pem
```

* `cert.pem`

```
-----BEGIN CERTIFICATE-----
MIICyjCCAbICCQDrpZYh8et7yTANBgkqhkiG9w0BAQsFADAnMQswCQYDVQQGEwJV
UzELMAkGA1UECAwCQ0ExCzAJBgNVBAcMAlNGMB4XDTE4MTExMjIwNDEwNVoXDTE4
MTIxMjIwNDEwNVowJzELMAkGA1UEBhMCVVMxCzAJBgNVBAgMAkNBMQswCQYDVQQH
DAJTRjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJnIdgpml8+xk+Oj
1RGMCyJ1P15RiM6rdtszT+DFBg893Lqsjoyd5YgwELLz0Ux8nviG4L5OXOujEpAP
2cQBxTSLQjBELBZY9q0Qky3+2ewqV6lSfcXrcf/JuDJGR5K8HSqwNG35R3WGnZ+O
JhY0Dmx06IAs/FF8gP88zTQ8M7zuaThkF8MaF4sWPf6+texQwjzk4rewknGBFzar
9wFxVwNCyDD6ewIYPtgDxdJ1bwBVoX3KKKXm8GStl/Zva0aEtbSq/161J4VbTro2
dxArMPKzxjD6NLyF59UNs7vbzyfiw/Wq7BJzU7Kued5KdGt0bEiyWZYO+EvvxGmE
1pHfqysCAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAavj4CA+7XFVHbwYMbK3c9tN/
73hkMvkAZWix5bfmOo0cNRuCeJnRIX+o6DmusIc8eXJJJV/20+zoSvUwlsLDPXoN
+c41GfIiEUSaSdSBtETMy8oPga718nIwAvNgYiUHXnV3B0nLYBUpYSnsD00/6VXG
xZUIEVBd7Ib5aRwmK8U5drxoWaBoG5qdvH9iapwTrCcPsRjsLBq7Iza2oBORGlfF
CjqiW2+KJzwRiTQj70yceniGVHM+VSpFYCLJ0mXeyLfITy7joqxr4AGYz+EhpLuf
iDpYDNYlr0JDVQqogskWjrnWOh0YcIJKgVtiTh2HDM5TdQgeXg4wv5IqLok0Tw==
-----END CERTIFICATE-----
```

```bash
$ vault kv put kv-v1/prod/cert/mysql cert=@cert.pem
```

## Save multiple values

```bash
$ vault kv put kv-v1/dev/config/mongodb \
        url=foo.example.com:35533 \
        db_name=users \
        username=admin password=passw0rd
```

```bash
$ tee mongodb.json <<EOF
{
    "url": "foo.example.com:35533",
    "db_name": "users",
    "username": "admin",
    "password": "pa$$w0rd"
}
EOF
```

```bash
$ vault kv put kv-v1/dev/config/mongodb @mongodb.json
```

## Generate a token for apps

```bash
$ tee apps-policy.hcl <<EOF
# Read-only permit
path "kv-v1/eng/apikey/Google" {
  capabilities = [ "read" ]
}

# Read-only permit
path "kv-v1/prod/cert/mysql" {
  capabilities = [ "read" ]
}
EOF
```

```bash
$ vault policy write apps apps-policy.hcl
$ APPS_TOKEN=$(vault token create -policy="apps" -field=token)
```

## Read the Secrets

```bash
$ vault kv get kv-v1/<PATH>
```

Read the secret at the path `kv-v1/eng/apikey/Google` using the apps token.

```bash
$ VAULT_TOKEN=$APPS_TOKEN vault kv get kv-v1/eng/apikey/Google
```

Read only the key field at the path `kv-v1/eng/apikey/Google`.

```bash
$ VAULT_TOKEN=$APPS_TOKEN vault kv get -field=key kv-v1/eng/apikey/Google
$ VAULT_TOKEN=$APPS_TOKEN vault kv get -field=cert kv-v1/prod/cert/mysql
```