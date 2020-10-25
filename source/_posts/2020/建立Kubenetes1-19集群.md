---
title: 建立Kubenetes1.19集群--kubeadm方式
date: 2020-10-18 10:32:27
tags: [kubernetes,k8s]
---

目前最新版本的Kubenetes集群是1.19，本次打算在Ubuntu 20.04 上安装集群，使用官网的Kubeadm 方式进行安装

<!-- more -->

# 环境规划：

| IP                          |              |          |
| --------------------------- | ------------ | -------- |
| uk8s-master(192.168.85.200) | Ubuntu 20.04 | VM 2C 4G |
| uk8s-node1(192.168.85.201)  | Ubuntu 20.04 | VM 2C 4G |
|                             |              |          |

# 基础环境准备


## 配置静态IP和主机名

修改静态IP

```
sudo vim /etc/netplan/00-installer-config.yaml
```

```shell
network:
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.85.200/24]
      optional: true
      gateway4: 192.168.85.2
      nameservers:
              addresses: [114.114.114.114]
  version: 2
```

网络生效

```
sudo netplan apply
```

修改主机名

```
sudo hostnamectl set-hostname uk8s-master
```

设置网络转发规则

```
echo "
net.bridge.bridge-nf-call-ip6tables=1
net.bridge.bridge-nf-call-iptables=1
net.ipv4.ip_forward=1
net.ipv4.conf.all.forwarding=1
net.ipv4.neigh.default.gc_thresh1=4096
net.ipv4.neigh.default.gc_thresh2=6144
net.ipv4.neigh.default.gc_thresh3=8192
net.ipv4.neigh.default.gc_interval=60
net.ipv4.neigh.default.gc_stale_time=120
" >> /etc/sysctl.conf
sysctl -p
```

关闭swap  可以编辑/etc/fstab   #注掉swap

```
swapoff -a
```



## 修改国内更新源

1、备份配置文件

```
sudo cp -a /etc/apt/sources.list /etc/apt/sources.list.bak
```

2、修改sources.list

```
sudo sed -i "s@http://.*archive.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list
sudo sed -i "s@http://.*security.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list
```

3、更新索引

```
sudo apt-get update
```
## 开放端口
Kubernate 集群会在不同的节点开放一些端口，以提供正常的集群内部状态监听，可以参照[官网文档](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)

ubuntu 20.04 默认情况下ufw服务是没有开启的，先设置好ssh 的端口然后开启ufw服务
```
sudo ufw allow ssh
sudo ufw enable
#查看ufw 状态
sudo ufw status

```

Master 节点开放下面端口，执行下面命令

```
sudo ufw allow 6443/tcp
sudo ufw allow 6443/upd
sudo ufw allow 2379/tcp
sudo ufw allow 2380/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10251/tcp
sudo ufw allow 10252/tcp

sudo ufw reload

```

```
sudo ufw allow 6443/tcp
sudo ufw allow 6443/upd

sudo ufw reload

```

## 安装Docker
Ubuntu 安装Docker 还是非常简单的，详细信息可以参照[官方文档](https://docs.docker.com/engine/install/ubuntu/)

# 安装Kubeadm

使用华为云

```
sudo apt-get update && apt-get install -y apt-transport-https

sudo cat <<EOF > /etc/apt/sources.list.d/kubernetes.list 
deb https://mirrors.huaweicloud.com/kubernetes/apt/ kubernetes-xenial main
EOF

sudo curl -s https://mirrors.huaweicloud.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

sudo apt update
sudo apt install -y kubeadm kubelet kubectl
```

使用阿里云

```
sudo apt-get update && apt-get install -y apt-transport-https

sudo curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 

sudo cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

执行kubeadm version 显示出版本号

```
kubeadm version

kubeadm version: &version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.3", GitCommit:"1e11e4a2108024935ecfcb2912226cedeafd99df", GitTreeState:"clean", BuildDate:"2020-10-14T12:47:53Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}
```

# 创建Kubernetes集群

执行kubeadm init 命令创建集群

```
kubeadm init --apiserver-advertise-address=192.168.85.200 --image-repository=registry.aliyuncs.com/google_containers --kubernetes-version v1.19.0 --service-cidr=10.1.0.0/16 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=all
```

执行上面命令会出现下面的内容，表示集群创建成功，创建过程会下载镜像。

```
W1018 09:08:13.976547   54088 configset.go:348] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.19.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local uk8s-master] and IPs [10.1.0.1 192.168.85.200]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost uk8s-master] and IPs [192.168.85.200 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost uk8s-master] and IPs [192.168.85.200 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 19.513130 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.19" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node uk8s-master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node uk8s-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: ooa5ht.hjhimxdj8mp1owr8
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.85.200:6443 --token ooa5ht.hjhimxdj8mp1owr8 \
    --discovery-token-ca-cert-hash sha256:6be4b027c3f102a1f9a2bf802569fca6d0d982dda4f0fef35b729cbacf6d402b 
```

在Master 节点执行完成下面三行命令，就可以使用kubectl 集合访问集群.

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
```
# 显示集群状态
kubectl get nodes

NAME          STATUS     ROLES    AGE     VERSION
uk8s-master   NotReady      master   5d21h   v1.19.3
```

在Node1节点上执行加入集群的命令，将节点加入到集群中
```
# 要从上面复制过来，每次创建集群的Token都是不一样的。

kubeadm join 192.168.85.200:6443 --token ooa5ht.hjhimxdj8mp1owr8 \
    --discovery-token-ca-cert-hash sha256:6be4b027c3f102a1f9a2bf802569fca6d0d982dda4f0fef35b729cbacf6d402b
```
显示下面信息表示节点加入集群成功。

```
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
W1018 07:22:19.659240   31810 kubelet.go:205] detected "cgroupfs" as the Docker cgroup driver, the provided value "systemd" in "KubeletConfiguration" will be overrided
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```
再次显示集群的状态
```
kubectl get nodes

NAME          STATUS     ROLES    AGE     VERSION
uk8s-master   NotReady      master   5d21h   v1.19.3
uk8s-node1    NotReady   <none>   5d21h   v1.19.3
```
现在看两个节点都是NoeReady状态，是因为没有安装CNI网络组件的原因,从calico官网下载部署的yaml文件,然后部署到集群中。

```
wget https://docs.projectcalico.org/manifests/calico.yaml

kubectl applpy -f calico.yaml

```
需要下载一些镜像，如果不能科学上网就修改calio.yaml里面的镜像地址，过一会时间再次显示集群信息就显示正常了

```
image: calico/cni:v3.15.1
这个位置修改成aliyun的镜像地址

```


问题：

1、如果kubeadm join 的token 失效或者丢失，可以使用下面命令重新生成

```
sudo kubeadm token create --print-join-command
```

2、国内源有时候也不是很稳定，阿里、华为、清华找一些切换一下

