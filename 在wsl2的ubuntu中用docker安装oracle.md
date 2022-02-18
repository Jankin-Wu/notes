# 在wsl2的ubuntu中用docker安装oracle

## 换国内镜像源

1. 将系统源文件复制一份备用

```sh
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

2. 用 vi 编辑器打开源文件

```sh
sudo vi /etc/apt/sources.list
# 然后直接输入49dd，就可以清除所有内容了，然后输入i就可以进行编辑了
```

3. 粘贴以下内容

```sh
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

4. 更新系统

```sh
sudo apt-get -y update && sudo apt-get -y upgrade
```

## 安装docker

```sh
#添加GPG密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 
#设置存储库位置
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 
#执行安装命令
apt-get install -y docker-ce
```

## 拉取镜像

```sh
docker pull oracleinanutshell/oracle-xe-11g
```

## 拷贝数据

```sh
# 先运行容器
docker run -d -p 1521:1521 -e ORACLE_ALLOW_REMOTE=true -p 8081:8080 --name oracle oracleinanutshell/oracle-xe-11g
# 在宿主机中创建挂载文件夹
mkdir -p /usr/local/docker/volumes/
# 将容器中的数据拷贝到宿主机
docker cp oracle:/u01/app/oracle/  /usr/local/docker/volumes/
```

## 重新运行容器

```sh
# 删除刚才运行的oracle容器，运行以下命令
docker run -d -p 1521:1521 -e ORACLE_ALLOW_REMOTE=true -p 8081:8080 -v /usr/local/docker/volumes/oracle/:/u01/app/oracle/ --restart=always --privileged=true --name oracle oracleinanutshell/oracle-xe-11g
```

## 设置静态IP

```sh
# 编写以下脚本，并在组策略添加登录启动脚本
@echo off
setlocal enabledelayedexpansion

wsl -u root service docker start | findstr "Starting Docker" > nul
if !errorlevel! equ 0 (
    echo docker start success
    :: set wsl2 ip
    wsl -u root ip addr | findstr "192.168.169.2" > nul
    if !errorlevel! equ 0 (
        echo wsl ip has set
    ) else (
        wsl -u root ip addr add 192.168.169.2/28 broadcast 192.168.169.15 dev eth0 label eth0:1
        echo set wsl ip success: 192.168.169.2
    )

    :: set windows ip
    ipconfig | findstr "192.168.169.1" > nul
    if !errorlevel! equ 0 (
        echo windows ip has set
    ) else (
        netsh interface ip add address "vEthernet (WSL)" 192.168.169.1 255.255.255.240
        echo set windows ip success: 192.168.169.1
    )
)
pause
```

## 端口映射

```SH
# netsh interface portproxy add v4tov4 listenport=[win10端口] listenaddress=0.0.0.0 connectport=[虚拟机的端口] connectaddress=[虚拟机的ip]
netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=80 connectaddress=172.29.41.233
```



## 使用Navicat连接

服务名：XE

账号：system

密码：oracle

## 可能遇到的问题

1. 能ping 通，但ssh连接不上

   解决办法：使用 “/usr/sbin/ssh -d” 命令调试，显示 “connect to host 172.25.95.30 port 22: Connection refused”，使用 “service sshd start” 显示 “ssh:unrecognized service”，则可能是没装ssh服务，输入以下命令解决：

   ```sh
   # 安装ssh服务器
   sudo apt install openssh-server
   # 安装ssh客户端
   sudo apt install openssh-client
   # 配置ssh客户端，去掉PasswordAuthentication yes前面的#号，保存退出
   # 配置ssh服务器，把PermitRootLogin prohibit-password改成PermitRootLogin yes，保存退出。
   sudo vi /etc/ssh/ssh_config
   # 重启ssh服务
   sudo /etc/init.d/ssh restart
   ```

   

