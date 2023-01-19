+++
title = "config k8s in archlinux"
date = 2023-01-19 09:28:42

[taxonomies]
categories=["k8s"]
tags=["k8s"]
+++

## Install packages
### Install necessary packages

```bash
pacman -S --needed kubeadm kubelet kubectl
```

### Install extra useful packages
```bash
pacman -S --needed helm kustomize k9s
```

## Generate containerd default configuration

```bash
# Generate default configuration
mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' \
	/etc/containerd/config.toml
sed -i 's/k8s.gcr.io/registry.cn-hangzhou.aliyuncs.com\/google_containers/g' \
	/etc/containerd/config.toml
```

## (Optional) config crictl

```
crictl config --set  runtime-endpoint=unix:///run/containerd/containerd.sock
```
## Enable and start services

```bash
systemctl enable --now kubelet.service containerd.service
```

- better to reboot system beacuse for iptables-nft modules to load
- please disable swap for kubelet to run


## Run kubeadm

### pull images from aliyun

```bash
kubeadm config images pull \
    --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers
```

### kubeadm init

```bash
POD_CIDR="10.244.0.0/16"
DNS_NAME=tkit-vbox
kubeadm init \
    --apiserver-advertise-address 0.0.0.0 \
	--apiserver-cert-extra-sans $DNS_NAME \
    --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
    --pod-network-cidr=$POD_CIDR
```

## Untaint control plane
```bash
kubectl taint node --all  node-role.kubernetes.io/control-plane:NoSchedule-
```

## Config k8s network

```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

## Load balancer

### Metallb

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
```
Config files:

IpAddressPool.yml:
```yml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: ip-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.123.50-192.168.123.90
```
L2Advertisement.yml:
```yml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advertisement
  namespace: metallb-system
```
**Apply config**
```bash
kubectl apply -f IpAddressPool.yml
kubectl apply -f L2Advertisement.yml
```


## Ingress

### Kong
```bash
helm repo add kong https://charts.konghq.com
helm repo update
helm install --create-namespace --namespace kong kong kong/kong
```

## k8s gateway api

### Kong

install gateway api crds

```bash
kubectl kustomize "https://github.com/kubernetes-sigs/gateway-api/config/crd?ref=v0.5.1" | kubectl apply -f -
```

install ingress controller with helm

values.yaml
```yaml
ingressController.env.feature_gates: GatewayAlpha=true
```

add gateway class add gateway
```bash
echo "apiVersion: gateway.networking.k8s.io/v1beta1
kind: GatewayClass
metadata:
  name: kong
  annotations:
    konghq.com/gatewayclass-unmanaged: 'true'
spec:
  controllerName: konghq.com/kic-gateway-controller
" | kubectl apply -f -

```

```bash
echo "apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: kong
spec:
  gatewayClassName: kong
  listeners:
  - name: proxy
    port: 80
    hostname: kong.example
    protocol: HTTP
  - name: proxy-ssl
    port: 443
    hostname: kong.example
    protocol: HTTPS
" | kubectl apply -f -

```
example http route
```bash
 echo "apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: echo
  annotations:
    konghq.com/strip-path: 'true'
spec:
  parentRefs:
  - group: gateway.networking.k8s.io
    kind: Gateway
    name: kong
  hostnames:
  - kong.example
  rules:
  - backendRefs:
    - group: ""
      kind: Service
      name: echo
      port: 80
      weight: 1
    matches:
    - path:
        type: PathPrefix
        value: /echo
" | kubectl apply -f -

```

## cert-manager