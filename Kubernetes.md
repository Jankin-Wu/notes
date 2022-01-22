# Kubernetes

## Kubernetes简介

​		kubernetes的本质是一组服务器集群，它可以在集群的每个节点上运行特定的程序，来对节点中的容器进行管理。它的目的就是实现资源管理的自动化，主要提供了如下的主要功能：
+ **自我修复：**一旦某一个容器崩溃，能够在1秒中左右迅速启动新的容器
+ **弹性伸缩：**可以根据需要，自动对集群中正在运行的容器数量进行调整
+ **服务发现：**服务可以通过自动发现的形式找到它所依赖的服务
+ **负载均衡：**如果一个服务起动了多个容器，能够自动实现请求的负载均衡
+ **版本回退：**如果发现新发布的程序版本有问题，可以立即回退到原来的版本
+ **存储编排：**可以根据容器自身的需求自动创建存储卷

## Kubernetes 组件

![img](http://oss.jankinwu.com/img/7464841-1513d229dedd9024.png)

> **Master 组件**

**• kube-apiserver** Kubernetes API，集群的统一入口，各组件协调者，以RESTful API提供接口服务，所有对象资源的增删改查和监听操作都交给APIServer    处理后再提交给etcd存储。

**• kube-controller-manager** 处理集群中常规后台任务，一个资源对应一个控制器，而ControllerManager 就是负责管理这些控制器的。

**• kube-scheduler** 根据调度算法为新创建的Pod选择一个Node节点，可以任意部署,可以部署在同一个节点上,也可以部署在不同的节点上。
**• etcd** 分布式键值存储系统。用于保存集群状态数据，比如Pod、Service等对象信息。

> **Node组件**

**• kubelet** kubelet是Master在Node节点上的Agent，管理本机运行容器的生命周期，比如创 建容器、Pod挂载数据卷、下载secret、获取容器和节点状态等工作。kubelet将每个Pod转换成一组容器。

**• kube-proxy** 在Node节点上实现Pod网络代理，维护网络规则和四层负载均衡工作。

**• docker或rocket** 容器引擎，运行容器。

## Kubernetes核心概念

+ **Master：**集群控制节点，每个集群需要至少个master-节点负责集群的管控
+ **Node：**工作负载节点，由master分配容器到这些node工作节点上，然后node节点上的dockerf负责容器的运行
+ **Pod**：kubernetes的最小控制单元，容器都是运行在pod中的，一个pod中可以有1个或者多个容器
+ **Controller：**控制器，通过它来实现对pod的管理，比如启动pod、停止pod、伸缩pod的数量等等
+ **Service：**pod对外服务的统一入口，下面可以维护者同一类的多个pod 
+ **Label：**标签，用于对pod进行分类，同一类pod会拥有相同的标签
+ **NameSpace：**命名空间，用来隔离pod的运行环境

## 环境搭建

### 环境初始化

**1. 检查操作系统版本**

```sh
# centos版本在7.5及以上
cat /etc/redhat-release
```

**2. 设置主机名解析**

```sh
# 为了方便后面集群节点间的直接调用，在这配置一下主机名解析，企业中推荐使用内部DNS服务器
vim /etc/hosts

xxx.xxx.xxx.xxx master
xxx.xxx.xxx.xxx node1
xxx.xxx.xxx.xxx node2
```

**3. 时间同步**

​		kubernetes要求集群中的节点时间必须精确一致，这里直接使用chronyd服务从网络同步时间。企业中建议配置内部的时间同步服务器。

```sh
#启动chronyd服务
systemctl start chronyd
#设置chronyd服务开机自启
systemctl enable chronyd
#chronyd服务启动稍等几秒钟，就可以使用date命令验证时间了
date
```

**4. 关闭防火墙**

```sh
# 关闭防火墙
systemctl stop firewalld
# 设置开机禁用防火墙
systemctl disable firewalld
```

**5. 禁用 selinux**

selinux 是 linux 系统下的一个安全服务，如果不关闭它，在安装集群中能产生各种各样的奇葩问题。

```sh
vim /etc/selinux/config
# 设为disable，设置完后重启系统
SELINUX=disable
# 查看selinux是否关闭
getenforce
```

**6. 禁用swap分区**

```sh
# 进入fstab注释掉swap后，重启系统
vim /etc/fstab
```

**7. 修改linux内核参数**

```sh
# 修改linux内核参数，添加网桥过滤和地址转发功能
# 编辑/etc/sysctl.d/kubernetes.conf文件
vim /etc/sysctl.d/kubernetes.conf
# 添加如下配置
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
# 重新加载配置
sysctl -p
# 加载网桥过滤模块
modprobe br_netfilter
# 查看网桥过滤模块是否加载成功
lsmod | grep br_netfilter
```

**8. 配置ipvs功能**

​		在kubernetest中service有两种代理模型，一种是基于iptables的，一种是基于ipvs的两者比较的话，ipvs的性能明显要高一些，但是如果要使用它，需要手动载入ipvs模块。 

```sh
# 安装 ipset 和 ipvsadm
yum install ipset ipvsadm -y
# 添加需要加载的模块写入脚本文件
# 高版本的centos内核nf_conntrack_ipv4替换为nf_conntrack
cat <<EOF > /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
# 为脚本文件添加执行权限
chmod +x /etc/sysconfig/modules/ipvs.modules
# 执行脚本文件
/bin/bash /etc/sysconfig/modules/ipvs.modules
# 查看对应的模块是否加载成功
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

**9. 重启服务器**

```sh
# 上面步骤完成后，需要重启服务器
reboot
```

### 安装docker

```sh
# 1、切换镜像源
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
# 2、查看当前镜像源中支持的docker版本
yum list docker-ce --showduplicates
# 3、安装特定版本的docker-ce
# 必须制定--setopt=obsoletes=0，否则yum会自动安装更高版本
yum install --setopt=obsoletes=0 docker-ce-18.06.3.ce-3.el7 -y

# 4、添加一个配置文件
#Docker 在默认情况下使用Vgroup Driver为cgroupfs，而Kubernetes推荐使用systemd来替代cgroupfs
mkdir /etc/docker
cat <<EOF> /etc/docker/daemon.json
{
"exec-opts": ["native.cgroupdriver=systemd"],
"registry-mirrors": ["https://kn0t2bca.mirror.aliyuncs.com"]
}
EOF
# 5、启动dokcer
systemctl restart docker
# 6、将docker设为开机自启
systemctl enable docker
```

### 安装Kubernetes组件

```sh
# 1、由于kubernetes的镜像在国外，速度比较慢，这里切换成国内的镜像源
# 2、编辑/etc/yum.repos.d/kubernetes.repo,添加下面的配置
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgchech=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
			https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

# 3、安装kubeadm、kubelet和kubectl
yum install --setopt=obsoletes=0 kubeadm-1.17.4-0 kubelet-1.17.4-0 kubectl-1.17.4-0 -y

# 4、配置kubelet的cgroup
#编辑/etc/sysconfig/kubelet, 添加下面的配置
KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
KUBE_PROXY_MODE="ipvs"

# 5、设置kubelet开机自启
systemctl enable kubelet
```

### 准备集群镜像

```sh
# 在安装kubernetes集群之前，必须要提前准备好集群需要的镜像，所需镜像可以通过下面命令查看
kubeadm config images list

# 下载镜像
# 此镜像kubernetes的仓库中，由于网络原因，无法连接，下面提供了一种替换方案
images=(
kube-apiserver:v1.17.4
kube-controller-manager:v1.17.4
kube-scheduler:v1.17.4
kube-proxy:v1.17.4
pause:3.1
etcd:3.4.3-0
coredns:1.6.5
)
# 将镜像从阿里云仓库下载下来，并把前缀改为k8s.gcr.io
for imageName in ${images[@]};do
	docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
	docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
	docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName 
done
```



### 集群初始化

> 下面的操作只需要在master节点上执行即可

```sh
# 创建集群
# 将apiserver-advertise-address改为master节点的ip地址
kubeadm init \
	--apiserver-advertise-address=192.168.90.100 \
	--image-repository registry.aliyuncs.com/google_containers \
	--kubernetes-version=v1.17.4 \
	--service-cidr=10.96.0.0/12 \
	--pod-network-cidr=10.244.0.0/16
# 创建必要文件
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

