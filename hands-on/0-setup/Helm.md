# Setup

## Install Vault

* https://www.vaultproject.io/docs/platform/k8s/helm/run

```bash
$ helm repo add hashicorp https://helm.releases.hashicorp.com
# Disable persistent storage for dev
$ helm install vault hashicorp/vault --set server.dataStorage.enabled=false
```

# Initialize and unseal

```bash
# Initialize
$ kubectl exec -ti vault-0 -- vault operator init

# Unseal
$ kubectl exec -ti vault-0 -- vault operator unseal # ... Unseal Key 1
$ kubectl exec -ti vault-0 -- vault operator unseal # ... Unseal Key 2
$ kubectl exec -ti vault-0 -- vault operator unseal # ... Unseal Key 3
```


Unseal Key 1: 2/bl3nFojiG2+Oim/3xgEdhVY26Xcr5471QorjjnbehC
Unseal Key 2: KdWjF0Q+/sP/kdSu6z4vYZNPdkC2EE65Izj1+Wml1Mgk
Unseal Key 3: OmACktKxLnUCUc3FjcCfUwe2S3xKYUn7eP1ROscO0UG3
Unseal Key 4: FZo0klxKiO0Z8GtZvA3r6EDKSeGHZcvJIhIJ9QwFGbHY
Unseal Key 5: MSsxokN1rF5rwxADsfzBwCkYyDRLJiyYp8YIhULe8mz4

Initial Root Token: s.Do3Qr6gRRUQvNiQ3HHFPBHLN


