# Policies

## Create an ACL policy

Save it as `read.hcl`.

```bash
path "<PATH>/*" {
  capabilities = ["read", "list", "create", "patch", "update"]
}
```

Create policy with `read.hcl`

```bash
$ vault policy write <POLICY_NAME> <PATH/TO/HCL_FILE>

$ vault policy write ro-policy read.hcl
```

## Update Entity Policy

```bash
$ vault write identity/entity name="<ENTITY_NAME>" policies="<POL1>,<POL2>"

$ vault write identity/entity name="user-0-entity" policies="ro-policy"
```