+++
authors = ["Jun Wu"]
title = "Set up Kubernetes Cluster on Hyper-V - 2024"
date = "2024-06-01"
description = "a guide to use set up a Kubernetes cluster."
tags = [
    "kubernetes"
]
categories = [
    "infra",
    "devops"
]
series = ["proxy"]
+++

## Prerequisites

- 3 or more VM running Ubuntu 24 LTS on Hyper-V

## Install kubeadm & kubectl

### Install containerd-io

```sh
wget https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz
```

```sh
wget https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
install -m 755 runc.amd64 /usr/local/sbin/runc
```

```sh
wget https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
sudo mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.0.tgz
```

then edit config 

```sh
sudo mkdir -p /etc/containerd/
sudo containerd config default | sudo tee /etc/containerd/config.toml
```


```sh
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo cp containerd.service /usr/local/lib/systemd/system/containerd.service

sudo mkdir -p /etc/systemd/system/containerd.service.d
sudo vi /etc/systemd/system/containerd.service.d/http-proxy.conf
```

```
[Service]
Environment="HTTP_PROXY=http://192.168.137.1:7890"
Environment="HTTPS_PROXY=http://192.168.137.1:7890"
Environment="NO_PROXY=localhost"
```

```sh
sudo systemctl daemon-reload
sudo systemctl restart containerd
sudo systemctl start containerd
sudo systemctl enable containerd
sudo systemctl status containerd
```


```sh
sudo rm -rf /etc/systemd/system/containerd.service.d/http-proxy.conf
sudo systemctl daemon-reload
sudo systemctl restart containerd
```

```sh
sudo systemctl stop containerd
```

try with Redis


### Install nerdctl

```sh
wget https://github.com/containerd/nerdctl/releases/download/v1.7.6/nerdctl-1.7.6-linux-amd64.tar.gz
tar Cxzvvf /usr/local/bin nerdctl-1.7.6-linux-amd64.tar.gz
```

```sh
ctr images pull docker.io/library/redis:alpine
nerdctl run -d -p 6379:6379 --name my-redis docker.io/library/redis:alpine
```


## Enable IP Forwarding

```sh
sudo sh -c "echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf"
sudo sysctl -p
```

## Install kubeadm

### Disable swap

```sh
sudo vi /etc/fstab
```

comment out anything like `/swapfile swap swap defaults 0 0`.


### Install by apt

```sh
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Set up Control Plane

## check service status

```sh
sudo systemctl status containerd
sudo systemctl status kubelet
sudo journalctl -u kubelet -xe
```


```sh
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.5.253
```


if everything goes well, you will see

```sh
kubeadm join 192.168.5.253:6443 --token s7t4mk.psxv48qwj0tqoinf \
        --discovery-token-ca-cert-hash sha256:6aefb9813d6f99d344282d57d3766771c13ebe9ddbd930a5b6a0f4ecfd42fdf0
```

## Install network add on - Flannel

```sh
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

## Worker node

If you missed the token that was displayed when you initialized the cluster, you can look up the currently valid token with the following command

```sh
kubeadm token list
```

if the token expired, you can generate a new one.

```sh
kubeadm token create
```

then print the hash of CA cert

```sh
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

so you can run the following on worker node

```sh
sudo su -
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

and then label the worker node

```sh
kubectl label node kube-worker-1 node-role.kubernetes.io/worker=
```

## Install load balancer - MetalLB

>*Note*
> more details can be found from -> https://metallb.universe.tf/installation/


```sh
kubectl edit configmap -n kube-system kube-proxy
```

and set 

```yaml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

install MetalLB 

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
```

set IP range for load balancer

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.5.200-192.168.5.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
```

try to verify the external ip for a service with type `LoadBalancer`

```sh
kubectl create deploy nginx --image nginx:latest
```

```sh
kubectl expose deploy nginx --port 80 --type LoadBalancer
```

then you can test the nginx via external ip

```sh
curl http://192.168.5.200
```

if successed, we can delete the nginx

```sh
kubectl delete deploy nginx
kubectl delete svc nginx
```


## Install Helm

Helm is one of MUST for Kubernetes, most poplular components provide Helm Packages to install on Kubernetes.

install it in your control plane is a suggested way:

```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Install and config Storage

Another ubuntu server is reqiured to host as a Network File System.


```sh
sudo apt update
sudo apt install nfs-kernel-server
```

```sh
sudo mkdir -p /srv/nfs/kubedata
sudo chown nobody:nogroup /srv/nfs/kubedata
sudo chmod 777 /srv/nfs/kubedata
sudo mount /dev/mapper/ubuntu--vg-data /srv/nfs/kubedata
```

```sh
sudo vi /etc/exports
```

```sh
/srv/nfs/kubedata *(rw,sync,no_subtree_check,no_root_squash)
```

```sh
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
sudo systemctl enable nfs-kernel-server
sudo systemctl restart nfs-kernel-server
sudo systemctl status nfs-kernel-server
```

try to verify it, please go back to your Kubernetes contronl plane, install nfs-common

### Install nfs-common in all nodes (both control plane & worker nodes)

```sh
sudo apt update
sudo apt install nfs-common
```

```sh
sudo mkdir -p /mnt/nfs-test
sudo mount 192.168.5.189:/srv/nfs/kubedata /mnt/nfs-test2
```


### Install nfs-subdir-external-provisioner

```sh
$ helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
$ helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner  --set nfs.server=192.168.5.189 --set nfs.path=/srv/nfs/kubedata

helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --create-namespace \
  --namespace nfs-provisioner \
  --set nfs.server=192.168.5.189 \
  --set nfs.path=/srv/nfs/kubedata
```

create a test `PersistentVolumeClaim`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-test
  labels:
    storage.k8s.io/name: nfs
    storage.k8s.io/part-of: kubernetes-complete-reference
    storage.k8s.io/created-by: ssbostan
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-client
  resources:
    requests:
      storage: 20Mi
---
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - mountPath: "/mnt/nfs"
          name: nfs-storage
  volumes:
    - name: nfs-storage
      persistentVolumeClaim:
        claimName: nfs-test     
```


## Install Ingress Controller

### Install Traefik

```sh
helm repo add traefik https://traefik.github.io/charts
helm repo update
kubectl create ns traefik-v2
# Install in the namespace "traefik-v2"
helm install --namespace=traefik-v2 \
    traefik traefik/traefik
```

then you can run cmd to get the external ip of the ingress class

```sh
kubectl -n traefik-v2 get svc traefik  -o jsonpath="{.status.loadBalancer.ingress[*].ip}"
```

you will see the ip

```sh
192.168.5.200
```

install a http demo service

```sh
kubectl create deploy whoami --image traefik/whoami
kubectl expose deploy whoami --port 80 --type LoadBalancer
```

try to use the IngressRoute by default 

```yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: myingressroute
  namespace: default

spec:
  entryPoints:
    - web

  routes:
  - match: Host(`whoami.kube-traefik-lb.com`)
    kind: Rule
    services:
    - name: whoami
      port: 80
```

then you can edit `hosts` file to test the domain and the ingress route rule

```sh
192.168.5.200 whoami.kube-traefik-lb.com
```

open the url `whoami.kube-traefik-lb.com` in your Chrome browser.

you will see 

```
Hostname: whoami-98d7579fb-65d58
IP: 127.0.0.1
IP: ::1
IP: 10.244.1.5
IP: fe80::4ce9:87ff:fee3:7439
RemoteAddr: 10.244.1.4:37008
GET / HTTP/1.1
Host: whoami.pikaqiu.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 10.244.2.0
X-Forwarded-Host: whoami.pikaqiu.com
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: traefik-5bfcbb8567-bb4kp
X-Real-Ip: 10.244.2.0
```

### Config HTTPS & TLS 

```sh
helm upgrade traefik traefik/traefik --namespace traefik-v2 -f values.yaml
```




## Manage cluster

get admin config

```sh
cat ./.kube/config
```


### Install Metrics Server

```sh
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```


### Install prometheus and grafana

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

## Check cluster

ensure cluster is not using any proxy, it will break the internal dns:


```sh
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

```sh
sudo rm -rf /etc/systemd/system/containerd.service.d/http-proxy.conf
sudo systemctl daemon-reload
sudo systemctl restart containerd
```

remove any proxy config from `.env` part.

### list of containers

```sh
sudo nerdctl --namespace k8s.io ps -a

sudo nerdctl logs  --namespace k8s.io 07a7ecfe4bae
```

## Reset Kubernetes

```sh
sudo kubeadm reset
rm -rf  $HOME/.kube/config
sudo rm -rf /etc/cni/net.d
```

