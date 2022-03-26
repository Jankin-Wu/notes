# Jenkins

## 持续集成

**持续集成的好处**

1. 降低风险，由于持续集成不断去构建，编译和测试，可以很早期发现问题，所以修复的代价就少；

2. 对系统健康持续检查，减少发布风险带来的问题；

3. 减少重复性工作；

4. 持续部署，提供可部署单元包；

5. 持续交付可供使用的版本；

6. 增强团队信心；

## Jenkins 介绍

Jenkins 是一款流行的开源持续集成（Continuous Integration）工具，广泛用于项目开发，具有自动

化构建、测试和部署等功能。官网： http://jenkins-ci.org/。

**Jenkins的特征**

+ 开源的Java语言开发持续集成工具，支持持续集成，持续部署。

+ 易于安装部署配置：可通过yum安装,或下载war包以及通过docker容器等快速实现安装部署，可

方便web界面配置管理。

+ 消息通知及测试报告：集成RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知，生

成JUnit/TestNG测试报告。

+ 分布式构建：支持Jenkins能够让多台计算机一起构建/测试。

+ 文件识别：Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar等。

+ 丰富的插件支持：支持扩展插件，你可以开发适合自己团队使用的工具，如git，svn，maven，

docker等。

**Jenkins 自动化部署实现原理**

![img](http://oss.jankinwu.com/img/858186-20190804162611917-80438542.png)

## Jenkins 安装和持续集成环境配置

### gitlab 安装

#### 使用 docker 安装

**运行 gitlab 容器**

```sh
docker run -d  -p 1443:443 -p 1080:80 -p 222:22 --name gitlab --restart always -v /usr/local/docker/gitlab/config:/etc/gitlab -v /usr/local/docker/gitlab/logs:/var/log/gitlab -v /usr/local/docker/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
```

**配置**

```sh
vim /usr/local/docker/gitlab/config/gitlab.rb
# 添加以下配置
# 配置http协议所使用的访问地址,不加端口号默认为80
external_url 'http://192.168.199.231'
# 配置ssh协议所使用的访问地址和端口
gitlab_rails['gitlab_ssh_host'] = '192.168.199.231'
gitlab_rails['gitlab_shell_ssh_port'] = 222
```

**重启gitlab容器**

```sh
docker restart gitlab
```

### Jenkins 安装

#### 使用 docker 安装

**1. 创建 Jenkins 挂载目录并授予权限**

```sh
//创建目录
mkdir -p /usr/local/docker/jenkins_home
//授权权限
chmod 777 /usr/local/docker/jenkins_home
```

**2. 启动 Jenkins 容器**

```sh
# 最好先去dockerhub上找到最新版本，拉取指定版本号，否则容易出现插件安装失败的问题
docker run -d -p 8081:8080 -p 50000:50000 -v /usr/local/docker/jenkins_home:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkins jenkins/jenkins:2.340
```

**3. 打开浏览器访问**

http://localhost:8081

**4. 获取并输入admin账户密码**

```sh
vim /usr/local/docker/jenkins_home/secrets/initialAdminPassword
```

#### 在 linux 上直接安装

**1. 安装JDK**

```sh
# Jenkins需要依赖JDK，所以先安装JDK1.8
yum install java-1.8.0-openjdk* -y
# 安装目录为：/usr/lib/jvm
```

**2. 获取jenkins安装包**

下载页面：https://jenkins.io/zh/download/

安装文件：jenkins-2.190.3-1.1.noarch.rpm

**3. 把安装包上传到192.168.66.101服务器，进行安装**

```sh
rpm -ivh jenkins-2.190.3-1.1.noarch.rpm
```

**4. 修改Jenkins配置**

```sh
vi /etc/syscofifig/jenkins

# 修改内容如下：
JENKINS_USER="root"
JENKINS_PORT="8888"
```

**5. 启动Jenkins**

```shell
systemctl start jenkins
```

**6. 打开浏览器访问**

http://localhost:8888

**7. 获取并输入admin账户密码**

```shell
cat /var/lib/jenkins/secrets/initialAdminPassword
```
### Jenkins 配置

#### 插件管理

**1. 跳过插件安装**

​		因为Jenkins插件需要连接默认官网下载，速度非常慢，而且经过会失败，所以我们暂时先跳过插件安

装。

![image-20220323153736298](http://oss.jankinwu.com/img/image-20220323153736298.png)

![image-20220323155427930](http://oss.jankinwu.com/img/image-20220323155427930.png)

**2. 创建管理员用户**

![image-20220323155538634](http://oss.jankinwu.com/img/image-20220323155538634.png)

![image-20220323155650994](http://oss.jankinwu.com/img/image-20220323155650994.png)

![image-20220323155741369](http://oss.jankinwu.com/img/image-20220323155741369.png)

**3. 修改 Jenkins 插件下载源**

Jenkins->Manage Jenkins->Manage Plugins，点击Available，这样做是为了把Jenkins官方的插件列表下载到本地。

![image-20220323160539887](http://oss.jankinwu.com/img/image-20220323160539887.png)

> 修改地址文件，替换为国内插件地址。

```shell
# linux 安装
cd /var/lib/jenkins/updates
# docker 安装，因为做了映射，直接在宿主机挂载的jenkins_home中修改
cd /usr/local/docker/jenkins_home/updates

# 换成清华镜像库
# 使用docker安装的jenkins，如没有“default.json”文件，则访问“http://localhost:8081/restart”后再执行以下步骤
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
```

> 重启 Jenkins

访问 `http://localhost:1080/restart`，地址为部署的地址后加`/restart`

> 修改Update Site

在Manage Plugins点击Advanced，把Update Site改为国内插件下载地址：

https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

然后点击submit

![image-20220326143556516](http://oss.jankinwu.com/img/image-20220326143556516.png)

> 重启 Jenkins

访问 `http://localhost:1080/restart`，地址为部署的地址后加`/restart`

**4. 下载中文汉化插件**

![image-20220326144801927](http://oss.jankinwu.com/img/image-20220326144801927.png)

#### 用户权限管理

**1. 安装Role-based Authorization Strategy插件**

![image-20220326160427660](http://oss.jankinwu.com/img/image-20220326160427660.png)

**2. 授权策略切换为"Role-Based Strategy"**

![image-20220326160554973](http://oss.jankinwu.com/img/image-20220326160554973.png)

![image-20220326160838186](http://oss.jankinwu.com/img/image-20220326160838186.png)

**3. 创建角色**

系统管理 -> Manage and Assign Roles -> Manage Roles

Global roles（全局角色）：管理员等高级用户可以创建基于全局的角色 

Project roles（项目角色）：针对某个或者某些项目的角色

Slave roles（奴隶角色）：节点相关的权限

**4. 创建用户**

系统管理 -> Manage Users -> 新建用户

**5. 给用户分配角色**

系统管理 -> Manage and Assign Roles -> Assign Roles

在 `User/group to add`中输入需要分配角色的用户，勾选相应角色，保存。

#### 凭证管理

​		凭据可以用来存储需要密文保护的数据库密码、Gitlab密码信息、Docker私有仓库密码等，以便

Jenkins可以和这些第三方的应用进行交互

##### 安装Credentials Binding插件

![image-20220326165544931](http://oss.jankinwu.com/img/image-20220326165544931.png)

​		安装插件后，在安全菜单会显示凭证管理选项，在这里管理所有凭证。

![image-20220326170246949](http://oss.jankinwu.com/img/image-20220326170246949.png)

可以添加的凭证有5种：

+ Username with password：用户名和密码

+ SSH Username with private key： 使用SSH用户和密钥

+ Secret fifile：需要保密的文本文件，使用时Jenkins会将文件复制到一个临时目录中，再将文件路径设置到一个变量中，等构建结束后，所复制的Secret fifile就会被删除。

+ Secret text：需要保存的一个加密的文本串，如钉钉机器人或Github的api token

+ Certifificate：通过上传证书文件的方式

##### 安装Git插件和Git工具

为了让Jenkins支持从Gitlab拉取源码，需要安装Git插件以及在jenkins服务器上安装Git工具。

![image-20220326171446661](http://oss.jankinwu.com/img/image-20220326171446661.png)

```sh
# 使用linux安装jenkins，需要jenkins所在的服务器安装git，docker 版的容器已经安装，所以不需要再安装
yum install git -y
```

