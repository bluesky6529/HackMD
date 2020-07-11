# 使用Rancher部屬簡單的k8s集群

## 前言
架構使用rke+helm3+certmanagerv0.15.0部屬


## Using tools
1Master 3Worker (主要在master 操作 master 到每一台機器都需埋ssh key)
env: ubuntu 18.04
每一台都需裝docker

## master安裝
1.rke
```shell=
wget https://github.com/rancher/rke/releases/download/v1.0.8/rke_linux-amd64
chmod 755 rke_linux-amd64
cp rke_linux-amd64 /bin/rke
```
2.kubectl
```shell=
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl
chmod 755 kubectl
cp kubectl /bin/kubectl
```
3. helm
```shell=
wget https://get.helm.sh/helm-v3.2.1-linux-amd64.tar.gz
tar zxvf helm-v3.2.1-linux-amd64.tar.gz
mv linux-amd64/helm .
chmod 755 helm 
cp helm /bin/helm
```
---

### 部屬步驟
1. 建立 rancher-cluster.yml
```
nodes:
  - address: 10.10.10.160
    internal_address: 10.10.10.160
    user: root
    role: [controlplane,etcd]

  - address: 10.10.10.161
    internal_address: 10.10.10.161
    user: root
    role: [worker]
  - address: 10.10.10.162
    internal_address: 10.10.10.162
    user: root
    role: [worker]
  - address: 10.10.10.163
    internal_address: 10.10.10.163
    user: root
    role: [worker]

services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 12h

ssh_key_path: ~/.ssh/id_rsa
```
2. 使用rke 安裝cluster
```shell=
rke up --config ./rancher-cluster.yml
mkdir /root/.kube
cp kube_config_rancher-cluster.yml /root/.kube/config
```
3. helm安裝rancher
添加helm repo
```
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
```
建立名為cattle-system的namespace(命名空間)
```
kubectl create namespace cattle-system
```
添加cert-manager crd
```
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.0/cert-manager.crds.yaml
```
建立cert-manager namespace 
```
kubectl create namespace cert-manager
```
添加/安裝 cert-manager v0.15.0
```
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.15.0
```
確認是否安裝成功
```
kubectl get pods --namespace cert-manager
```
添加cluster issuer
* 先建立 clusterissuer.yaml

```
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: 123@gmail.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: nginx
---
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: 123@gmail.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx

```
* kubectk apply -f clusterissuer.yaml

裝好certmanager後，再裝rancher
```
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org(your domain name) \ 
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org
```
確認是否安裝成功
```
kubectl -n cattle-system rollout status deploy/rancher
```

