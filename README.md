# astratex-rke2

## Master

```
sudo mkdir -p /etc/rancher/rke2
```

```
sudo vi /etc/rancher/rke2/config.yaml
```

```
token: xxx
tls-san:
  - k8s-api.astratex.sikademo.com
node-taint:
  - "CriticalAddonsOnly=true:NoExecute"
kube-apiserver-arg:
  - "oidc-issuer-url=https://sso.sikalabs.com/realms/sikademo"
  - "oidc-client-id=astratex"
  - --oidc-username-claim=email
  - --oidc-groups-claim=groups
disable:
  - rke2-ingress-nginx
```

```
curl -sfL https://get.rke2.io | sudo sh -
```

```
sudo systemctl enable rke2-server.service --now
```

```
sudo cat /etc/rancher/rke2/rke2.yaml
```

## Worker

```
sudo mkdir -p /etc/rancher/rke2
```

```
sudo vi /etc/rancher/rke2/config.yaml
```

```
token: xxx
server: https://k8s-api.astratex.sikademo.com:9345
```

```
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sudo sh -
```

```
sudo systemctl enable rke2-agent.service --now
```

## Ingress Nginx

```
helm upgrade --install \
  ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --create-namespace \
  --namespace ingress-nginx \
  --set controller.service.type=ClusterIP \
  --set controller.ingressClassResource.default=true \
  --set controller.kind=DaemonSet \
  --set controller.hostPort.enabled=true \
  --set controller.metrics.enabled=true \
  --set controller.config.use-proxy-protocol=false \
  --wait
```

## Cert Manager

```
helm upgrade --install \
  cert-manager cert-manager \
  --repo https://charts.jetstack.io \
  --create-namespace \
  --namespace cert-manager \
  --set installCRDs=true \
  --wait
```

## Cluster Issuer

```
curl -XPOST https://auth.acme-dns.io/register
```

```
{
  "username": "19e9fbe3-9f63-4366-b2f1-634967ba6d90",
  "password": "xxx",
  "fulldomain": "9079a471-6184-4a05-bf45-c3f26dbe5acd.auth.acme-dns.io",
  "subdomain": "9079a471-6184-4a05-bf45-c3f26dbe5acd",
  "allowfrom": []
}
```

```
kubectl apply -f clusterissuer.yml
```

## Example App

```
helm upgrade --install \
  hello-world hello-world \
  --repo https://helm.sikalabs.io \
  --create-namespace \
  --namespace hello-world \
  --set host=hello.astratex.sikademo.com \
  --set replicas=1 \
  --set TEXT= \
  --wait
```

## Get Certificates using Lego

```
lego --email ondrejsika@ondrejsika.com --dns cloudflare \
  --domains atx.sikademo.com \
  --domains '*.atx.sikademo.com' \
  run
```

## Kubeconfig with OIDC

Apply RBAC

```
kubectl apply -f rbac_sso_admin.yml
```

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJlakNDQVIrZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWtNU0l3SUFZRFZRUUREQmx5YTJVeUxYTmwKY25abGNpMWpZVUF4TnpFNU16QTNNamczTUI0WERUSTBNRFl5TlRBNU1qRXlOMW9YRFRNME1EWXlNekE1TWpFeQpOMW93SkRFaU1DQUdBMVVFQXd3WmNtdGxNaTF6WlhKMlpYSXRZMkZBTVRjeE9UTXdOekk0TnpCWk1CTUdCeXFHClNNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJDMVF5VlJYZ3V5OUNnQWNiMEY4N0JKeUJ0V2FLdHhINVFrVUpmWmMKM3cwYkRYQUkvUkhRa3p6UnE1R216TTVLOU1oR0JQOXMwUzNPNjFtazdBOGlFY3VqUWpCQU1BNEdBMVVkRHdFQgovd1FFQXdJQ3BEQVBCZ05WSFJNQkFmOEVCVEFEQVFIL01CMEdBMVVkRGdRV0JCU254bmVTWUp1RlNwUzA2dE9NClNxVTU5dm1ISURBS0JnZ3Foa2pPUFFRREFnTkpBREJHQWlFQTUremRFNXFGMGlzQ29kNHZmWkJwMm1GTkZvMVgKMk9aYkNRVTRKQkhicmkwQ0lRRE9JcVplZm0yalBRS2xrWjZ0cm00L0VtK3g2RFJRK1ZFaVAxSFVXK2lybGc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://k8s-api.astratex.sikademo.com:6443
  name: astratex
contexts:
- context:
    cluster: astratex
    user: astratex
  name: astratex
current-context: astratex
kind: Config
preferences: {}
users:
- name: astratex
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: kubectl
      args:
      - oidc-login
      - get-token
      - --oidc-issuer-url=https://sso.sikalabs.com/realms/sikademo
      - --oidc-client-id=astratex
      - --oidc-client-secret=xxx
```
