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
yum install --setopt=obsoletes=0 kubeadm-1.23.2-0 kubelet-1.23.2-0 kubectl-1.23.2-0 -y

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
kube-apiserver:v1.23.2
kube-controller-manager:v1.23.2
kube-scheduler:v1.23.2
kube-proxy:v1.23.2
pause:3.6
etcd:3.5.1-0
coredns:1.8.6
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
# 加--v=6 可以查看更详细日志
kubeadm init \
	--apiserver-advertise-address=192.168.90.100 \
	--image-repository registry.aliyuncs.com/google_containers \
	--kubernetes-version=v1.23.2 \
	--service-cidr=10.96.0.0/12 \
	--pod-network-cidr=10.244.0.0/16
	--v=6
# 创建必要文件
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
下面的操作只需要在node节点上执行即可

```sh
# 初始化master后会生成以下命令
kubeadm join 192.168.0.100:6443 --token awk15p.t6bamck54w69u4s8 \
    --discovery-token-ca-cert-hash sha256:a94fa09562466d32d29523ab6cff122186f1127599fa4dcd5fa0152694f17117 
```



### 安装网络插件

```sh
# 只在master节点操作即可
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### 使用配置文件启动fannel

```sh
kubectl apply -f kube-flannel.yml
```


### 卸载 kubernetes

```sh
kubeadm reset -f
modprobe -r ipip
lsmod
rm -rf ~/.kube/
rm -rf /etc/kubernetes/
rm -rf /etc/systemd/system/kubelet.service.d
rm -rf /etc/systemd/system/kubelet.service
rm -rf /usr/bin/kube*
rm -rf /etc/cni
rm -rf /opt/cni
rm -rf /var/lib/etcd
rm -rf /var/etcd
yum clean all
yum remove kube*
```

## 集群测试

### 创建一个nginx服务

```sh
kubectl create deployment nginx  --image=nginx:1.14-alpine
```

### 暴露端口

```sh
kubectl expose deploy nginx  --port=80 --target-port=80  --type=NodePort
```

### 查看服务

```sh
kubectl get pod
kubectl get svc
```

## 资源管理

### 资源管理介绍

​		在kubernetes中，所有的内容都抽象为资源，用户需要通过操作资源来管理kubernetes。

​		kubernetes的本质上就是一个集群系统，用户可以在集群中部署各种服务，所谓的部署服务，其实就是在kubernetes集群中运行一个个的容器，并将指定的程序跑在容器中。

​		kubernetes的最小管理单元是pod而不是容器，所以只能将容器放在`Pod`中，而kubernetes一般也不会直接管理Pod，而是通过`Pod控制器`来管理Pod的。

​		Pod可以提供服务之后，就要考虑如何访问Pod中服务，kubernetes提供了`Service`资源实现这个功能。

​		当然，如果Pod中程序的数据需要持久化，kubernetes还提供了各种`存储`系统。

![img](http://oss.jankinwu.com/img/image-20200406225334627.png)

### 资源管理方式

+ 命令式对象管理：直接使用命令去操作kubernetes资源

```sh
kubectl run nginx-pod --image=nginx:1.17.1 --port=80
```

+ 命令式对象配置：通过命令配置和配置文件去操作kubernetes资源

```shell
kubectl create/patch -f nginx-pod.yaml
```

+ 声明式对象配置：通过apply命令和配置文件去操作kubernetes资源

```sh
kubectl apply -f nginx-pod.yaml
```

| 类型           | 操作对象 | 适用环境 | 优点           | 缺点                             |
| -------------- | -------- | -------- | -------------- | -------------------------------- |
| 命令式对象管理 | 对象     | 测试     | 简单           | 只能操作活动对象，无法审计、跟踪 |
| 命令式对象配置 | 文件     | 开发     | 可以审计、跟踪 | 项目大时，配置文件多，操作麻烦   |
| 声明式对象配置 | 目录     | 开发     | 支持目录操作   | 意外情况下难以调试               |

### 命令式对象管理

**kubectl命令**

kubectl是kubernetes集群的命令行工具，通过它能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。kubectl命令的语法如下：

```sh
# comand：指定要对资源执行的操作，例如create、get、delete
# type：指定资源类型，比如deployment、pod、service
# name：指定资源的名称，名称大小写敏感
# flags：指定额外的可选参数
kubectl [command] [type] [name] [flags]
```

```sh
# 查看所有pod
kubectl get pod 

# 查看某个pod
kubectl get pod pod_name

# 查看某个pod,以yaml格式展示结果
kubectl get pod pod_name -o yaml
```

**资源类型**

kubernetes中所有的内容都抽象为资源，可以通过下面的命令进行查看：

```sh
kubectl api-resources
```
常用资源：

<table>
    <tr>
        <th>资源分类</th>
        <th>资源名称</th>
        <th>缩写</th>
        <th>资源作用</th>
    </tr>
    <tr>
    	<td rowspan="2">集群级别资源</td>
        <td>nodes</td>
        <td>no</td>
        <td>集群组成部分</td>
    </tr>
    <tr>
    	<td>namespaces</td>
        <td>ns</td>
        <td>隔离Pod</td>
    </tr>
    <tr>
    	<td>pod资源</td>
        <td>pods</td>
        <td>po</td>
        <td>装载容器</td>
    </tr>
    <tr>
    	<td rowspan="8">pod资源控制器</td>
        <td>replicationcontrollers</td>
        <td>rc</td>
        <td>控制pod资源</td>
    </tr>
    <tr>
    	<td>replicasets</td>
        <td>rs</td>
        <td>控制pod资源</td>
    </tr>
    <tr>
    	<td>deployments</td>
        <td>deploy</td>
        <td>控制pod资源</td>
    </tr>
    <tr>
    	<td>daemonsets</td>
        <td>ds</td>
        <td>控制pod资源</td>
    </tr>
    <tr>
    	<td>jobs</td>
        <td></td>
        <td>控制pod资源</td>
    </tr>
    <tr>
    	<td>cronjobs</td>
        <td>cj</td>
        <td>控制pod资源</td>
    </tr>
    <tr>
    	<td>horizontalpodautoscalers</td>
        <td>hpa</td>
        <td>控制pod资源</td>
    </tr>
    <tr>
    	<td>statefulsets</td>
        <td>sts</td>
        <td>控制pod资源</td>
    </tr>
    <tr>
    	<td rowspan="2">服务发现资源</td>
        <td>services</td>
        <td>svc</td>
        <td>统一pod对外接口</td>
    </tr>
    <tr>
    	<td>ingress</td>
        <td>ing</td>
        <td>统一pod对外接口</td>
    </tr>
    <tr>
    	<td rowspan="3">存储资源</td>
        <td>volumeattachments</td>
        <td></td>
        <td>存储</td>
    </tr>
    <tr>
    	<td>persistentvolumes</td>
        <td>pv</td>
        <td>存储</td>
    </tr>
    <tr>
    	<td>persistentvolumeclaims</td>
        <td>pvc</td>
        <td>存储</td>
    </tr>
    <tr>
    	<td rowspan="2">配置资源</td>
        <td>configmaps</td>
        <td>cm</td>
        <td>配置</td>
    </tr>
    <tr>
    	<td>secrets</td>
        <td></td>
        <td>配置</td>
    </tr>
</table>
**操作**

kubernetes允许对资源进行多种操作，可以通过--help查看详细的操作命令

```sh
kubectl --help
```

经常使用的操作有下面这些：

| 命令分类   | 命令         | 翻译                        | 命令作用                     |
| :--------- | :----------- | :-------------------------- | :--------------------------- |
| 基本命令   | create       | 创建                        | 创建一个资源                 |
|            | edit         | 编辑                        | 编辑一个资源                 |
|            | get          | 获取                        | 获取一个资源                 |
|            | patch        | 更新                        | 更新一个资源                 |
|            | delete       | 删除                        | 删除一个资源                 |
|            | explain      | 解释                        | 展示资源文档                 |
| 运行和调试 | run          | 运行                        | 在集群中运行一个指定的镜像   |
|            | expose       | 暴露                        | 暴露资源为Service            |
|            | describe     | 描述                        | 显示资源内部信息             |
|            | logs         | 日志输出容器在 pod 中的日志 | 输出容器在 pod 中的日志      |
|            | attach       | 缠绕进入运行中的容器        | 进入运行中的容器             |
|            | exec         | 执行容器中的一个命令        | 执行容器中的一个命令         |
|            | cp           | 复制                        | 在Pod内外复制文件            |
|            | rollout      | 首次展示                    | 管理资源的发布               |
|            | scale        | 规模                        | 扩(缩)容Pod的数量            |
|            | autoscale    | 自动调整                    | 自动调整Pod的数量            |
| 高级命令   | apply        | rc                          | 通过文件对资源进行配置       |
|            | label        | 标签                        | 更新资源上的标签             |
| 其他命令   | cluster-info | 集群信息                    | 显示集群信息                 |
|            | version      | 版本                        | 显示当前Server和Client的版本 |

下面以一个namespace / pod的创建和删除简单演示下命令的使用：

```shell
# 创建一个namespace
[root@master ~]# kubectl create namespace dev
namespace/dev created

# 获取namespace
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   21h
dev               Active   21s
kube-node-lease   Active   21h
kube-public       Active   21h
kube-system       Active   21h

# 在此namespace下创建并运行一个nginx的Pod
[root@master ~]# kubectl run pod --image=nginx:latest -n dev
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/pod created

# 查看新创建的pod
[root@master ~]# kubectl get pod -n dev
NAME  READY   STATUS    RESTARTS   AGE
pod   1/1     Running   0          21s

# 删除指定的pod
[root@master ~]# kubectl delete pod pod-864f9875b9-pcw7x
pod "pod" deleted

# 删除指定的namespace
[root@master ~]# kubectl delete ns dev
namespace "dev" deleted
```
### 命令式对象配置

命令式对象配置就是使用命令配合配置文件一起来操作kubernetes资源。

1. 创建一个nginxpod.yaml，内容如下：

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev

---

apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
  namespace: dev
spec:
  containers:
  - name: nginx-containers
    image: nginx:latest
```

2. 执行create命令，创建资源：

```powershell
[root@master ~]# kubectl create -f nginxpod.yaml
namespace/dev created
pod/nginxpod created
```

此时发现创建了两个资源对象，分别是namespace和pod

3. 执行get命令，查看资源：

```shell
[root@master ~]#  kubectl get -f nginxpod.yaml
NAME            STATUS   AGE
namespace/dev   Active   18s

NAME            READY   STATUS    RESTARTS   AGE
pod/nginxpod    1/1     Running   0          17s
```

这样就显示了两个资源对象的信息

4. 执行delete命令，删除资源：

```shell
[root@master ~]# kubectl delete -f nginxpod.yaml
namespace "dev" deleted
pod "nginxpod" deleted
```

此时发现两个资源对象被删除了

```
总结:
    命令式对象配置的方式操作资源，可以简单的认为：命令  +  yaml配置文件（里面是命令需要的各种参数）
```

### 声明式对象配置

声明式对象配置跟命令式对象配置很相似，但是它只有一个命令apply。

```shell
# 首先执行一次kubectl apply -f yaml文件，发现创建了资源
[root@master ~]#  kubectl apply -f nginxpod.yaml
namespace/dev created
pod/nginxpod created

# 再次执行一次kubectl apply -f yaml文件，发现说资源没有变动
[root@master ~]#  kubectl apply -f nginxpod.yaml
namespace/dev unchanged
pod/nginxpod unchanged
```

```powershell
总结:
    其实声明式对象配置就是使用apply描述一个资源最终的状态（在yaml中定义状态）
    使用apply操作资源：
        如果资源不存在，就创建，相当于 kubectl create
        如果资源已存在，就更新，相当于 kubectl patch
```

> kubectl怎么在node节点上运行

kubectl的运行是需要进行配置的，它的配置文件是$HOME/.kube，如果想要在node节点运行此命令，需要将master上的.kube文件复制到node节点上，即在master节点上执行下面操作：

```shell
scp  -r  HOME/.kube   node1: HOME/
```

> 使用推荐: 三种方式应该怎么用 ?

创建/更新资源 使用声明式对象配置 kubectl apply -f XXX.yaml

删除资源 使用命令式对象配置 kubectl delete -f XXX.yaml

查询资源 使用命令式对象管理 kubectl get(describe) 资源名称

## 各组件详解

### Namespace

Namespace是kubernetes系统中的一种非常重要资源，它的主要作用是用来实现**多套环境的资源隔离**或者**多租户的资源隔离**。

默认情况下，kubernetes集群中的所有的Pod都是可以相互访问的。但是在实际中，可能不想让两个Pod之间进行互相的访问，那此时就可以将两个Pod划分到不同的namespace下。kubernetes通过将集群内部的资源分配到不同的Namespace中，可以形成逻辑上的"组"，以方便不同的组的资源进行隔离使用和管理。

可以通过kubernetes的授权机制，将不同的namespace交给不同租户进行管理，这样就实现了多租户的资源隔离。此时还能结合kubernetes的资源配额机制，限定不同租户能占用的资源，例如CPU使用量、内存使用量等等，来实现租户可用资源的管理。

![image-20220125153235601](http://oss.jankinwu.com/img/image-20220125153235601.png)

kubernetes在集群启动之后，会默认创建几个namespace

```shell
[root@master ~]# kubectl  get namespace
NAME              STATUS   AGE
default           Active   45h     #  所有未指定Namespace的对象都会被分配在default命名空间
kube-node-lease   Active   45h     #  集群节点之间的心跳维护，v1.13开始引入
kube-public       Active   45h     #  此命名空间下的资源可以被所有人访问（包括未认证用户）
kube-system       Active   45h     #  所有由Kubernetes系统创建的资源都处于这个命名空间
```

#### 具体操作

**查看**

```shell
# 1 查看所有的ns  命令：kubectl get ns
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   45h
kube-node-lease   Active   45h
kube-public       Active   45h     
kube-system       Active   45h     

# 2 查看指定的ns   命令：kubectl get ns ns名称
[root@master ~]# kubectl get ns default
NAME      STATUS   AGE
default   Active   45h

# 3 指定输出格式  命令：kubectl get ns ns名称  -o 格式参数
# kubernetes支持的格式有很多，比较常见的是wide、json、yaml
[root@master ~]# kubectl get ns default -o yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2021-05-08T04:44:16Z"
  name: default
  resourceVersion: "151"
  selfLink: /api/v1/namespaces/default
  uid: 7405f73a-e486-43d4-9db6-145f1409f090
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
  
# 4 查看ns详情  命令：kubectl describe ns ns名称
[root@master ~]# kubectl describe ns default
Name:         default
Labels:       <none>
Annotations:  <none>
Status:       Active  # Active 命名空间正在使用中  Terminating 正在删除命名空间

# ResourceQuota 针对namespace做的资源限制
# LimitRange针对namespace中的每个组件做的资源限制
No resource quota.
No LimitRange resource.
```

**创建**

```shell
# 创建namespace
[root@master ~]# kubectl create ns dev
namespace/dev created
```

**删除**

```shell
# 删除namespace
[root@master ~]# kubectl delete ns dev
namespace "dev" deleted
```

**配置方式**

首先准备一个yaml文件：ns-dev.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

然后就可以执行对应的创建和删除命令了：

创建：kubectl create -f ns-dev.yaml

删除：kubectl delete -f ns-dev.yaml

### Pod

Pod是kubernetes集群进行管理的最小单元，程序要运行必须部署在容器中，而容器必须存在于Pod中。

Pod可以认为是容器的封装，一个Pod中可以存在一个或者多个容器。

![image-20220125160333743](http://oss.jankinwu.com/img/image-20220125160333743.png)

kubernetes在集群启动之后，集群中的各个组件也都是以Pod方式运行的。可以通过下面命令查看：

```shell
[root@master ~]# kubectl get pod -n kube-system
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-68g6v         1/1     Running   0          2d1h
kube-system   coredns-6955765f44-cs5r8         1/1     Running   0          2d1h
kube-system   etcd-master                      1/1     Running   0          2d1h
kube-system   kube-apiserver-master            1/1     Running   0          2d1h
kube-system   kube-controller-manager-master   1/1     Running   0          2d1h
kube-system   kube-flannel-ds-amd64-47r25      1/1     Running   0          2d1h
kube-system   kube-flannel-ds-amd64-ls5lh      1/1     Running   0          2d1h
kube-system   kube-proxy-685tk                 1/1     Running   0          2d1h
kube-system   kube-proxy-87spt                 1/1     Running   0          2d1h
kube-system   kube-scheduler-master            1/1     Running   0          2d1h
```
#### 具体操作
**创建并运行**

kubernetes没有提供单独运行Pod的命令，都是通过Pod控制器来实现的

```shell
# 命令格式： kubectl run (pod控制器名称) [参数] 
# --image  指定Pod的镜像
# --port   指定端口
# --namespace  指定namespace
[root@master ~]# kubectl run nginx --image=nginx:latest --port=80 --namespace dev 
deployment.apps/nginx created
```

**查看pod信息**

```shell
# 查看Pod基本信息
[root@master ~]# kubectl get pods -n dev
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          43s

# 查看Pod的详细信息
# 不加namespace名称默认查看default命名空间里的pod
[root@master ~]# kubectl describe pod nginx -n dev
Name:         nginx
Namespace:    dev
Priority:     0
Node:         node1/192.168.5.4
Start Time:   Wed, 08 May 2021 09:29:24 +0800
Labels:       pod-template-hash=5ff7956ff6
              run=nginx
Annotations:  <none>
Status:       Running
IP:           10.244.1.23
IPs:
  IP:           10.244.1.23
Controlled By:  ReplicaSet/nginx
Containers:
  nginx:
    Container ID:   docker://4c62b8c0648d2512380f4ffa5da2c99d16e05634979973449c98e9b829f6253c
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:485b610fefec7ff6c463ced9623314a04ed67e3945b9c08d7e53a47f6d108dc7
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 08 May 2021 09:30:01 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-hwvvw (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-hwvvw:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-hwvvw
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned dev/nginx-5ff7956ff6-fg2db to node1
  Normal  Pulling    4m11s      kubelet, node1     Pulling image "nginx:latest"
  Normal  Pulled     3m36s      kubelet, node1     Successfully pulled image "nginx:latest"
  Normal  Created    3m36s      kubelet, node1     Created container nginx
  Normal  Started    3m36s      kubelet, node1     Started container nginx
```

**访问Pod**

```shell
# 获取podIP
# -o wide，输出额外信息。对于Pod，将输出Pod所在的Node名
[root@master ~]# kubectl get pods -n dev -o wide
NAME    READY   STATUS    RESTARTS   AGE    IP             NODE    ... 
nginx   1/1     Running   0          190s   10.244.1.23   node1   ...

#访问POD
[root@master ~]# curl http://10.244.1.23:80
<!DOCTYPE html>
<html>
<head>
	<title>Welcome to nginx!</title>
</head>
<body>
	<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

**删除指定Pod**

```shell
# 删除指定Pod
[root@master ~]# kubectl delete pod nginx -n dev
pod "nginx" deleted

# 此时，显示删除Pod成功，但是再查询，发现又新产生了一个 
[root@master ~]# kubectl get pods -n dev
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          21s

# 这是因为当前Pod是由Pod控制器创建的，控制器会监控Pod状况，一旦发现Pod死亡，会立即重建
# 此时要想删除Pod，必须删除Pod控制器

# 先来查询一下当前namespace下的Pod控制器
[root@master ~]# kubectl get deploy -n  dev
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           9m7s

# 接下来，删除此PodPod控制器
[root@master ~]# kubectl delete deploy nginx -n dev
deployment.apps "nginx" deleted

# 稍等片刻，再查询Pod，发现Pod被删除了
[root@master ~]# kubectl get pods -n dev
No resources found in dev namespace.
```

**配置操作**

创建一个pod-nginx.yaml，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
spec:
  containers:
  - image: nginx:latest
    name: pod
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```

然后就可以执行对应的创建和删除命令了：

创建：kubectl create -f pod-nginx.yaml

删除：kubectl delete -f pod-nginx.yaml

### Label

Label是kubernetes系统中的一个重要概念。它的作用就是在资源上添加标识，用来对它们进行区分和选择。

Label的特点：

- 一个Label会以key/value键值对的形式附加到各种对象上，如Node、Pod、Service等等
- 一个资源对象可以定义任意数量的Label ，同一个Label也可以被添加到任意数量的资源对象上去
- Label通常在资源对象定义时确定，当然也可以在对象创建后动态添加或者删除

可以通过Label实现资源的多维度分组，以便灵活、方便地进行资源分配、调度、配置、部署等管理工作。

> 一些常用的Label 示例如下：
>
> - 版本标签："version":"release", "version":"stable"......
> - 环境标签："environment":"dev"，"environment":"test"，"environment":"pro"
> - 架构标签："tier":"frontend"，"tier":"backend"

标签定义完毕之后，还要考虑到标签的选择，这就要使用到Label Selector，即：

Label用于给某个资源对象定义标识

Label Selector用于查询和筛选拥有某些标签的资源对象

当前有两种Label Selector：

- 基于等式的Label Selector

  name = slave: 选择所有包含Label中key="name"且value="slave"的对象

  env != production: 选择所有包括Label中的key="env"且value不等于"production"的对象

- 基于集合的Label Selector

  name in (master, slave): 选择所有包含Label中的key="name"且value="master"或"slave"的对象

  name not in (frontend): 选择所有包含Label中的key="name"且value不等于"frontend"的对象

标签的选择条件可以使用多个，此时将多个Label Selector进行组合，使用逗号","进行分隔即可。例如：

name=slave，env!=production

name not in (frontend)，env!=production

##### 命令方式

```shell
# 为pod资源打标签
[root@master ~]# kubectl label pod nginx-pod version=1.0 -n dev
pod/nginx-pod labeled

# 为pod资源更新标签
[root@master ~]# kubectl label pod nginx-pod version=2.0 -n dev --overwrite
pod/nginx-pod labeled

# 查看标签
[root@master ~]# kubectl get pod nginx-pod  -n dev --show-labels
NAME        READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod   1/1     Running   0          10m   version=2.0

# 筛选标签
[root@master ~]# kubectl get pod -n dev -l version=2.0  --show-labels
NAME        READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod   1/1     Running   0          17m   version=2.0
[root@master ~]# kubectl get pod -n dev -l version!=2.0 --show-labels
No resources found in dev namespace.

#删除标签
[root@master ~]# kubectl label pod nginx-pod version- -n dev
pod/nginx-pod labeled
```

##### 配置方式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
  labels:
    version: "3.0" 
    env: "test"
spec:
  containers:
  - image: nginx:latest
    name: pod
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```

然后就可以执行对应的更新命令了：kubectl apply -f pod-nginx.yaml
