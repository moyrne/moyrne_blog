---
title: "Kubeadm 搭建K8s集群"
date: 2021-02-07T13:56:54+08:00 
draft: false
---

## 前置操作 (每台机器上都需要操作)

- 使用系统 Centos 7

~~~shell
cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
~~~

- 关闭防火墙

~~~shell
systemctl stop firewalld && systemctl disable firewalld
~~~

- 禁用SELINUX
~~~shell
vim /etc/selinux/config
~~~
~~~text
# 或者修改/etc/sysconfig/selinux
SELINUX=disabled
~~~

- 修改 k8s.conf

~~~shell
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
~~~

- 关闭 swap
~~~shell
swapoff -a
vim /etc/fstab
~~~
~~~text
# 注释掉以下字段
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
~~~
~~~shell
reboot
~~~

- 使用yum安装 `docker-ce` `docker-ce-selinux`

~~~shell
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 查看可安装的 docker-ce 版本
yum list docker-ce.x86_64 --showduplicates | sort -r
# Step 4 : 安装指定版本的Docker-CE
sudo yum -y --setopt=obsoletes=0 install docker-ce-[VERSION] docker-ce-selinux-[VERSION]
# Step 5: 开启Docker服务
sudo systemctl enable docker && systemctl start docker
~~~

- 安装成功后验证
~~~shell
docker version
~~~
~~~text
Client: Docker Engine - Community
 Version:           20.10.3
 API version:       1.40
 Go version:        go1.13.15
 Git commit:        48d30b5
 Built:             Fri Jan 29 14:34:14 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          19.03.10
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.10
  Git commit:       9424aeaee9
  Built:            Thu May 28 22:16:43 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.3
  GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
 runc:
  Version:          1.0.0-rc92
  GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
~~~

## 安装kubeadm，kubelet，kubectl

- 修改 yum 源

~~~shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
~~~

- 安装 `kubelet` `kubeadm` `kubectl`

~~~shell
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
~~~

- 创建 kubeadm init 初始化文件 (仅master)
~~~shell
vim kubeadm-init.yaml
~~~
~~~yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.20.0
imageRepository: registry.aliyuncs.com/k8sxio
controlPlaneEndpoint: "192.168.111.128:6443"
networking:
  serviceSubnet: "10.96.0.0/16"
  podSubnet: "10.100.0.1/16"
  dnsDomain: "192.168.111.128"
~~~

- 初始化 (仅master)
~~~shell
kubeadm init --config kubeadm-init.yaml
~~~
~~~shell
# 初始化完成后，会有以下提示出现
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
~~~

- 配置 kubectl (仅master), node上需要使用时，需要拷贝 config 文件

~~~shell
mkdir -p ~/.kube
sudo cp -i /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
~~~

- [警告] 当配置错误时可以使用
~~~shell
kubeadm reset
~~~
- 安装Pod Network

~~~text
# 比较知名的网络解决方案:
flannel
calico
canel
kube-router
.......
~~~

~~~shell
# 下载 至 本地
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 如果打不开也可以自行复制 https://github.com/coreos/flannel/blob/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
~~~

- 查看当前 Pod 状态

~~~shell
kubectl get pod -A -o wide
~~~

- 在 node 上执行 kubeadm init 完成时提示的命令

~~~shell
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
~~~

- 再次查看当前 Pod 状态

~~~shell
# -A == -all-namespace
kubectl get pod -A -o wide
~~~
~~~text
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
kube-system   coredns-68b9d7b887-2pswd         1/1     Running   1          22h   10.100.0.5        master   <none>           <none>
kube-system   coredns-68b9d7b887-7t42s         1/1     Running   1          22h   10.100.0.4        master   <none>           <none>
kube-system   etcd-master                      1/1     Running   1          22h   192.168.111.128   master   <none>           <none>
kube-system   kube-apiserver-master            1/1     Running   1          22h   192.168.111.128   master   <none>           <none>
kube-system   kube-controller-manager-master   1/1     Running   1          22h   192.168.111.128   master   <none>           <none>
kube-system   kube-flannel-ds-5h8jr            1/1     Running   1          22h   192.168.111.130   node2    <none>           <none>
kube-system   kube-flannel-ds-gzzhz            1/1     Running   1          22h   192.168.111.128   master   <none>           <none>
kube-system   kube-flannel-ds-pxh9j            1/1     Running   7          22h   192.168.111.129   node1    <none>           <none>
kube-system   kube-proxy-cxphf                 1/1     Running   1          22h   192.168.111.130   node2    <none>           <none>
kube-system   kube-proxy-hzh4k                 1/1     Running   1          22h   192.168.111.129   node1    <none>           <none>
kube-system   kube-proxy-kktrf                 1/1     Running   1          22h   192.168.111.128   master   <none>           <none>
kube-system   kube-scheduler-master            1/1     Running   1          22h   192.168.111.128   master   <none>           <none>
~~~