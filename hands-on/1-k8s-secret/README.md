# Kubernetes Secret

## 建立引擎掛載點

```bash
# 講師操作
kubectl port-forward -n vault pod/vault-0 8200:8200 --address 0.0.0.0
```

* 請利用 UI 建立對應掛載點
  * 請使用 key-value secret engine 掛載到 <user-id> 路徑 

* 建立 Service Account

```bash
kubectl create sa <sa-name>
```

## 進入 Vault

```bash
kubectl -n vault exec -it vault-0 -- sh

# in vault pod
export VAULT_TOKEN=<TOKEN>
```

## 寫入 Secret/Policy/Role

* `mount-entry` 為前個步驟地掛載點路徑 

* Secret

```bash
vault kv put <mount-entry>/<path> \
<key1>=<value1> \
<key2>=<value2>
```

* Policy

```bash
vault policy write <policy-name> -<<EOF
path \"<mount-entry>/*\" {
capabilities = [\"read\"]
}
EOF
```

* Role

```bash
vault write auth/kubernetes/role/<role-name> \
bound_service_account_names=<k8s-sa-name> \
bound_service_account_namespaces=<k8s-namespace> \
policies=<policy-name> \
ttl=1h
```

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: <spc-name>
  namespace: <user-id>
spec:
  provider: vault
  secretObjects:
  - secretName: <k8s-secret-name>
    type: Opaque
    data:
      - objectName: <obj1> # References dbUsername below
        key: <key-in-k8s-secret> # Key within k8s secret for this value
      - objectName: <obj2>
        key: <key-in-k8s-secret>
  parameters:
    roleName: '<role-name>' # match vault auth role name
    objects: |
      - objectName: "<obj1>"
        secretPath: "<mount-entry>/data/<path>"
        secretKey: "<key1>"
      - objectName: "<obj2>"
        secretPath: "<mount-entry>/data/<path>"
        secretKey: "<key2>"
```

## Sidecar Injector

```yaml
# https://medium.com/@verove.clement/inject-secret-in-application-from-vault-agent-injector-60a3fe71628e
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  namespace: api
  labels:
    app: api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/agent-inject-status: "update"
        vault.hashicorp.com/agent-inject-secret-app: "<mount-entry>/data/<path>"
        vault.hashicorp.com/agent-inject-template-app: |
          {{- with secret "<mount-entry>/data/<path>" -}}
          {{ range $k, $v := .Data.data }}
            export {{ $k }}={{ $v }}
          {{ end }}
          {{- end -}}
        vault.hashicorp.com/role: 'api'
    spec:
      serviceAccountName: api-serviceaccount
      restartPolicy: Always
      containers:
      - name: api
        image: verovec/golang-api-template:latest
        command: ["/bin/sh"]
        args: ["-c", ". /vault/secrets/app && ./app"]
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        env:
        - name: APP_ENV
          value: dev
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 30
          timeoutSeconds: 10
          periodSeconds: 10
          failureThreshold: 10
```

## Volume & Secret

```yaml
# https://pavan1999-kumar.medium.com/hashicvault-secrets-in-kubernetes-with-csi-driver-ec917d4a2672
# https://developer.hashicorp.com/vault/docs/platform/k8s/csi
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: <spc-name>
  namespace: <user-id>
spec:
  provider: vault
  secretObjects:
    - secretName: <k8s-secret-name>
      type: Opaque
      data:
        - objectName: <obj1> # References dbUsername below
          key: <key-in-k8s-secret> # Key within k8s secret for this value
        - objectName: <obj2>
          key: <key-in-k8s-secret>
  parameters:
    roleName: '<role-name>' # match vault auth role name
    objects: |
      - objectName: "<obj1>"
        secretPath: "<mount-entry>/data/<path>"
        secretKey: "<key1>"
      - objectName: "<obj2>"
        secretPath: "<mount-entry>/data/<path>"
        secretKey: "<key2>"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  labels:
    app: demo
  namespace: api
spec:
  selector:
    matchLabels:
      app: demo
  replicas: 1
  template:
    metadata:
      labels:
        app: demo
    spec:
      serviceAccountName: <k8s-sa-name>
      containers:
        - name: app
          image: nginx
          env:
          - name: <ENV_NAME>
            valueFrom:
              secretKeyRef:
                name: <k8s-secret-name>
                key: <key-in-k8s-secret>
          volumeMounts:
            - name: '<volume-name>'
              mountPath: '/mnt/secrets-store'
              readOnly: true
      volumes:
        - name: <volume-name>
          csi:
            driver: 'secrets-store.csi.k8s.io'
            readOnly: true
            volumeAttributes:
              secretProviderClass: '<spc-name>'
```