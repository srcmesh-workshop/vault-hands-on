# Vault

## Standalone

```bash
$ vault server -dev -dev-root-token-id root
$ export VAULT_ADDR=http://127.0.0.1:8200
$ export VAULT_TOKEN=root
```

## Helm Chart

* [Helm Chart Installation](Helm.md)