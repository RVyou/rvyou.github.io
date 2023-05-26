---
title: ubuntu 22.04 安装 kubernetes 1.26
author: rvyou
date: 2023-02-24 19:12:00
categories: [kubernetes,install]
tags: [kubernetes,install]
---

## 初始化 通用

```shell
#关闭默认dns服务
sudo systemctl status systemd-resolved
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
echo "nameserver 8.8.8.8"|sudo tee /etc/resolv.conf
#安装依赖
sudo apt-get update
sudo apt-get -y install apt-transport-https ipvsadm ipset sysstat conntrack libseccomp-dev ca-certificates curl gnupg  lsb-release  nfs-kernel-server


hostnamectl set-hostname <主机名字不能是localhost，三主节点不能重复>
#查看是否生效
hostnamectl status
#对应主机名写入hosts
echo "127.0.0.1   xxx"|sudo tee -a /etc/hosts

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo tee etc/modules-load.d/ipvs.conf << EOF
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack #内核小于4.18，把这行改成nf_conntrack_ipv4
EOF

sudo modprobe ip_vs
sudo modprobe ip_vs_rr
sudo modprobe ip_vs_wrr
sudo modprobe ip_vs_sh
sudo modprobe overlay
sudo modprobe br_netfilter

cat << EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.conf.all.proxy_arp         = 1
EOF

sudo sysctl --system
# 关闭 防火墙
sudo systemctl stop firewalld
sudo systemctl disable firewalld

# 关闭 SeLinux
sudo setenforce 0
sudo sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

# 关闭 swap
sudo swapoff -a
sudo cp /etc/fstab /etc/fstab_bak
sudo sed -ri 's/.*swap.*/#&/' /etc/fstab
```

```shell
#三台主节点的ip写入
cat <<EOF | sudo tee -a /etc/hosts
192.168.122.74 master1
192.168.122.110 master2
192.168.122.97 master3
EOF
```

### 安装 containerd

>  [官方文档](https://github.com/containerd/containerd/blob/main/docs/getting-started.md) 直接参考安装就好了

```shell
sudo mkdir /etc/containerd
containerd config default | sudo tee  /etc/containerd/config.toml

sudo sed -i "s#SystemdCgroup = false#SystemdCgroup = true#g" /etc/containerd/config.toml
#代理配置
#[plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
#  endpoint = ["https://hub-mirror.c.163.com"]
#这个也要修改
#sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
sudo systemctl restart containerd
#
```

### 安装  kubelet kubeadm kubectl

```shell
#这里使用清华大学源
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

sudo systemctl daemon-reload
sudo systemctl enable kubelet && sudo systemctl start kubelet
```

### kubeadm 安装 kubernetes

> 使用 kubeadm config print init-defaults --component-configs KubeletConfiguration 可以打印集群初始化默认的使用的配置

```yaml
cat <<EOF > ./kubeadm-config.yaml
---
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.122.74
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: master1 # 修改为第一台执行节点的hostname
  taints: null
---
controlPlaneEndpoint: 192.168.122.74:6443 # 新增控制平台地址
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: 1.26.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.100.0.0/16
scheduler: {}
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
failSwapOn: false
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
EOF
```

```shell
#下载镜像
sudo kubeadm config images pull --config=kubeadm-config.yaml
#初始化 master 其他master不需要
sudo kubeadm init --config=kubeadm-config.yaml --upload-certs
```

### 其他 master 加入集群

> kubeadm init phase upload-certs --upload-certs

```shell
kubeadm join 192.168.122.74:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:0806c301135d8a0daad21a7474e57888edd12e25de86645eb06e00ecb6e4d565 --control-plane
```

### node 节点加入集群

> 重新创建 token ：kubeadm token create --print-join-command

```shortcode
kubeadm join 192.168.122.74:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:0806c301135d8a0daad21a7474e57888edd12e25de86645eb06e00ecb6e4d56
```

### 安装网络插件calico

```shell
wget https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
wget https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml
sudo sed -i "s#192.168.0.0/16#10.100.0.0/16#" tigera-operator.yaml
sudo sed -i "s#192.168.0.0/16#10.100.0.0/16#" custom-resources.yaml
kubectl create -f tigera-operator.yaml
kubectl create -f custom-resources.yaml
```

```shell
#允许master 执行pod
kubectl taint nodes --all node-role.kubernetes.io/master-
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
# 恢复默认值
kubectl taint nodes NODE_NAME node-role.kubernetes.io/master=true:NoSchedule
```

## Reference

[kubeadm init | Kubernetes](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file)
