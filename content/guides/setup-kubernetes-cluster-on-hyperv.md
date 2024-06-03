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
sudo systemctl start containerd
sudo systemctl enable containerd
sudo systemctl status containerd
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
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.5.253
```


```sh
sudo kubeadm init --config=kubeadm-config.yaml
```

if everything goes well, you will see

```sh
kubeadm join 192.168.5.253:6443 --token s7t4mk.psxv48qwj0tqoinf \
        --discovery-token-ca-cert-hash sha256:6aefb9813d6f99d344282d57d3766771c13ebe9ddbd930a5b6a0f4ecfd42fdf0
```

### Install network add on - Flannel

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

## Check cluster

```sh
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

```sh
kubectl get pods -n kube-system
```

```sh
kubectl create deploy nginx --image nginx:latest
kubectl delete deploy nginx
```

### list of containers

```sh
sudo nerdctl --namespace k8s.io ps -a

sudo nerdctl logs  --namespace k8s.io 07a7ecfe4bae
```

## Reset kubeadm

```sh
sudo kubeadm reset
rm -rf  $HOME/.kube/config
sudo rm -rf /etc/cni/net.d
```

