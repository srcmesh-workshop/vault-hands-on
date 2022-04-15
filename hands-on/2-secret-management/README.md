# Secret Management

## Enable Key/Value Secret Engines

```bash
$ vault secrets enable -path="<MOUNT_PATH>" kv-v2

# Example
$ vault secrets enable -path="user-0" kv-v2
```

## Add Key/Value Secret

```bash
$ vault kv put <MOUNT_PATH>/<SECRET_NAME> <KEY>=<VALUE>

# Example
$ vault kv put user-0/foo bar=baz
```

## Update KV Secret

* Overwriting

```bash
$ vault kv put <MOUNT_PATH>/<SECRET_NAME> <KEY>=<VALUE>

# Example
$ vault kv put user-0/foo hello=world
```

* Without Overwriting

```bash
$ vault kv patch <MOUNT_PATH>/<SECRET_NAME> <KEY>=<VALUE>

# Example
$ vault kv patch user-0/foo hello=world
```

## Read KV Secret

```bash
$ vault kv get [-version=<VERSION>] <MOUNT_PATH>/<SECRET_NAME>

$ vault kv get user-0/foo
$ vault kv get -version=1 user-0/foo
$ vault kv get -version=2 user-0/foo
```

## Rollback KV Secret

```bash
$ vault kv rollback -version=<VERSION> <MOUNT_PATH>/<SECRET_NAME>

# Example
$ vault kv rollback -version=1 user-0/foo
```

## Add Custom Metadata

```bash
$ vault kv metadata put \
    -custom-metadata=<KEY1>=<VALUE_1> \
    -custom-metadata=<KEY2>=<VALUE_2> \
    <MOUNT_PATH>/<SECRET_NAME>
    
$ vault kv metadata put \
    -custom-metadata=b=a \
    user-0/foo
```

## Read Secret Metadata

```bash
$ vault kv metadata get <MOUNT_PATH>/<SECRET_NAME>

$ vault kv metadata get user-0/foo
```

## Specify the number of versions to keep

* Default KV keeps up to 10 versions.

```bash
$ vault kv metadata put -max-versions=<MAX_VERSIONS_TO_KEEP> <MOUNT_PATH>/<SECRET_NAME>

$ vault kv metadata put -max-versions=3 user-0/foo
```

## Check-And-Set Operations

Check whether CAS is enabled 

```bash
$ vault read <MOUNT_PATH>/config

$ vault read user-0/config
Key                     Value
---                     -----
cas_required            false
delete_version_after    0s
max_versions            0
```

Configure the secrets engine at path `MOUNT_PATH` to enable Check-And-Set.

```bash
$ vault write <MOUNT_PATH>/config cas_required=true
```

Configure the secret at path `<MOUNT_PATH>/<SECERT_NAME>` to enable Check-And-Set.

```bash
$ vault kv metadata put -cas-required=true <MOUNT_PATH>/<SECERT_NAME>
```

## Delete KV Data

```bash
$ vault kv delete [-versions=<VER1,VER2>] <MOUNT_PATH>/<SECRET_NAME>

$ vault kv delete user-0/foo
$ vault kv delete -versions=1 user-0/foo
```

It should be empty if you're trying to read `version 1` after we deleted it.

```bash
$ vault kv get -version=1 user-0/foo
```

## Automatic Data Deletion

```bash
$ vault kv metadata put -delete-version-after=<TIME> <MOUNT_PATH>/<SECRET_NAME>

$ vault kv metadata put -delete-version-after=10s user-0/foo
```

## Destroy KV Data (Permanently Delete Data)

```bash
$ vault kv destroy [-versions=<VER1,VER2>] <MOUNT_PATH>/<SECRET_NAME>

$ vault kv destroy user-0/foo
$ vault kv destroy -versions=1 user-0/foo
```