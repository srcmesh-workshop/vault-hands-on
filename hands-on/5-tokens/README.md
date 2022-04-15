# Tokens

## Prerequisite

ACL policy

* `acl.hcl`

```bash
path "auth/token/*" {
	capabilities = ["create", "read", "update", "delete", "list"]
}
```

Create policy

```bash
$ vault policy write <POLICY_NAME> acl.hcl
```

Update user entity with policy just created

```bash
$ vault write identity/entity name="<ENTITY_NAME>" policies="default,<POLICY_NAME>"
```

## Login

```bash
$ vault login -method=userpass -path=<MOUNT_PATH> username=<USERNAME>
$ export VAULT_TOKEN=<TOKEN>
```

## Create Token

```bash
$ vault token create -ttl=<TIME> -policy=<POLICY>
```

Verification

```bash
$ vault token lookup <TOKEN>
$ vault token lookup -accessor <TOKEN_ACCESSOR>
```

## Create token with use limit

```bash
$ vault token create -ttl=1h -use-limit=2 -policy=default
```

Verification

```bash
$ vault token lookup <TOKEN>
Key                 Value
---                 -----
accessor            eVB7UEnKSfmqcjQFF1lneJo8
...
num_uses            1
...

$ vault token lookup <TOKEN>
Key                 Value
---                 -----
accessor            eVB7UEnKSfmqcjQFF1lneJo8
...
num_uses            -1
...

$ vault token lookup <TOKEN>
Error looking up token: Error making API request.

URL: GET http://127.0.0.1:8200/v1/auth/token/lookup-self
Code: 403. Errors:

* permission denied
```

## Create Periodic Token

```bash
$ vault token create -policy=<POLICY> -period=<TIME>

$ vault token create -policy="default" -period=24h
Key                  Value
---                  -----
token                hvs.CAESIJKxT0hzANKWeTNIExhBeKqKPK5cPbr4OQEDl3z0Y1XpGh4KHGh2cy5sSVhJdEJOekZaazU0M1V6N0ZlRE9WMWI
token_accessor       LsCHHSMz28pn0kqqQpZJipEs
token_duration       24h
token_renewable      true
token_policies       ["default"]
identity_policies    []
policies             ["default"]
```

Check period

```bash
$ vault token lookup <TOKEN>

$ vault token lookup hvs.CAESIJKxT0hzANKWeTNIExhBeKqKPK5cPbr4OQEDl3z0Y1XpGh4KHGh2cy5sSVhJdEJOekZaazU0M1V6N0ZlRE9WMWI
Key                 Value
---                 -----
accessor            LsCHHSMz28pn0kqqQpZJipEs
creation_time       1649981383
creation_ttl        24h
display_name        token
entity_id           n/a
expire_time         2022-04-16T00:09:43.338252105Z
explicit_max_ttl    0s
id                  hvs.CAESIJKxT0hzANKWeTNIExhBeKqKPK5cPbr4OQEDl3z0Y1XpGh4KHGh2cy5sSVhJdEJOekZaazU0M1V6N0ZlRE9WMWI
issue_time          2022-04-15T00:09:43.338257304Z
meta                <nil>
num_uses            0
orphan              false
path                auth/token/create
period              24h
policies            [default]
renewable           true
ttl                 23h57m3s
type                service
```

## Create Orphan Token

```bash
$ vault token create -orphan
```

## Token Role

```bash
$ vault write auth/token/roles/<ROLE> \
    allowed_policies="<POLICY1>,<POLICY2>" \
    orphan=<BOOL> \
    period=<TIME>
    
$ vault write auth/token/roles/zabbix \
    allowed_policies="policy1, policy2, policy3" \
    orphan=true \
    period=8h
```

Create a token for `zabbix` role.

```bash
$ vault token create -role=zabbix
Key                  Value
---                  -----
token                hvs.CAESIKS1OgZwjFrV-cVDsRnpbWhUMtwWbdNZd0qjfHBMVnaPGh4KHGh2cy5wZ0FNYVpmb1FtR2NxZnUxdTFRcTZjYnU
token_accessor       2nlNdB9zURLvoCesVJramocw
token_duration       8h
token_renewable      true
token_policies       ["default" "policy1" "policy2" "policy3"]
identity_policies    []
policies             ["default" "policy1" "policy2" "policy3"]
```

## Renew service tokens

```bash
$ vault token create -ttl=15 -explicit-max-ttl=30 -policy=default
```

* `ttl`: Time-to-Live
* `explicit-max-ttl`: Max Time-to-Live

```bash
$ vault token renew <TOKEN>

# Extend the token's TTL to 60 seconds but cannot exceed explicit-max-ttl
$ vault token renew -increment=60 $(cat test_token.txt)
```

## Revoke service tokens

```text
parent_token (1 minute)
   |___ child_token (3 minutes)
   |___ orphan_token (3 minutes)
```

* Create a parent token
* Create a child token
* Create an orphan token