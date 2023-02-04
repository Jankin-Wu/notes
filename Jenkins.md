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

​		Jenkins 是一款流行的开源持续集成（Continuous Integration）工具，广泛用于项目开发，具有自动

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

**2. 安装Maven**

​		因为用到的docker镜像中不包含maven，所以要在宿主机中安装，通过文件挂载的方式提供调用

从官网（https://maven.apache.org/download.cgi）下载maven，并放到宿主机`/opt`目录下

![image-20220403234457185](http://oss.jankinwu.com/img/image-20220403234457185.png)

```sh
# 切换到要安装的文件夹
cd /opt
# 解压
tar -xzvf apache-maven-3.8.5-bin.tar.gz
# 配置 apache-maven-3.8.5/conf/settings.xml，添加阿里云maven镜像仓库
vim apache-maven-3.8.5/conf/settings.xml
# 在<mirrors>标签中添加以下配置
<mirror>
 <id>aliyunmaven</id>
 <mirrorOf>*</mirrorOf>
 <name>阿里云公共仓库</name>
 <url>https://maven.aliyun.com/repository/public</url>
</mirror>
# 添加环境变量
vi /etc/profile
# 在文件底部加上
export M2_HOME=/opt/apache-maven-3.8.5
export PATH=$PATH:${M2_HOME}/bin
# 保存并退出编辑，使用下面的命令让修改生效
source /etc/profile
# 验证Maven安装
mvn -version
```

如果宿主机没有安装Java，则如下图所示：

![image-20220404000225359](http://oss.jankinwu.com/img/image-20220404000225359.png)

**2. 启动 Jenkins 容器**

```sh
# 最好先去dockerhub上找到最新版本，拉取指定版本号，否则容易出现插件安装失败的问题
docker run -d -p 8081:8080 -p 50000:50000 --restart=always -v /usr/local/docker/jenkins_home:/var/jenkins_home -v /etc/localtime:/etc/localtime -v /opt/apache-maven-3.8.5:/usr/local/maven -e PHP_TZ="Asia/Shanghai" --name jenkins jenkins/jenkins:2.341-jdk8
```

**3. 打开浏览器访问**

http://localhost:8081

**4. 获取并输入admin账户密码**

```sh
vim /usr/local/docker/jenkins_home/secrets/initialAdminPassword
```

##### 在 Docker 容器中更新 Jenkins 版本

```sh
# 如果jenkins版本太老，可以在不影响jenkins项目的情况下，更新jenkins版本
# 1.以root用户进入jenkins容器
    docker exec -it -u root jenkins /bin/bash
# 2.在容器中下载jenkins的最新war包,]，如果嫌下载太慢也可以从官网（https://www.jenkins.io/download/）下载war包然后拷贝到挂载目录中
    wget http://mirrors.jenkins.io/war/latest/jenkins.war
# 3.查看容器中jenkins war包的位置，并备份原来的war包
    whereis jenkins
    cd /usr/share/jenkins
    cp jenkins.war jenkinsBAK.war
# 4.将/var/jenkins_home的包cp到/usr/share/jenkins下覆盖
    cp /var/jenkins_home/jenkins.war /usr/share/jenkins/
# 5.退出容器并重启
    exit
    docker restart jenkins
```



#### 在 linux 上直接安装

**1. 安装JDK**

```sh
# Jenkins需要依赖JDK，所以先安装JDK1.8
yum install java-1.8.0-openjdk* -y
# 安装目录为：/usr/lib/jvm
```

**2. 使用yum 安装jenkins**

官网下载页：https://jenkins.io/zh/download/

```sh
# 执行以下命令进行安装
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
yum install jenkins
```

**4. 修改Jenkins配置**

```sh
vi /etc/sysconfig/jenkins

# 修改内容如下：
JENKINS_USER="root"
JENKINS_PORT="8888"
# 注：这种方法试了并没有起作用，建议直接用docker run的时候映射端口
```

**5. 启动Jenkins**

```shell
systemctl start jenkins
```

**6. 打开浏览器访问**

​	http://localhost:8888

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

![image-20220403151401465](http://oss.jankinwu.com/img/image-20220403151401465.png)

![image-20220326160838186](http://oss.jankinwu.com/img/image-20220326160838186.png)

**3. 创建角色**

系统管理 -> Manage and Assign Roles -> Manage Roles

Global roles（全局角色）：管理员等高级用户可以创建基于全局的角色 

Project roles（项目角色）：针对某个或者某些项目的角色，最新版已改名为“Item roles”

Slave roles（奴隶角色）：节点相关的权限，最新版已改名为“Node roles”

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

+ Secret file：需要保密的文本文件，使用时Jenkins会将文件复制到一个临时目录中，再将文件路径设置到一个变量中，等构建结束后，所复制的Secret file就会被删除。

+ Secret text：需要保存的一个加密的文本串，如钉钉机器人或Github的api token

+ Certificate：通过上传证书文件的方式

### 安装 Git 插件和 Git 工具

为了让Jenkins支持从Gitlab拉取源码，需要安装Git插件以及在jenkins服务器上安装Git工具。

![image-20220326171446661](http://oss.jankinwu.com/img/image-20220326171446661.png)

```sh
# 使用linux安装jenkins，需要jenkins所在的服务器安装git，docker 版的容器已经安装，所以不需要再安装
yum install git -y
```

#### 使用 Git 用户密码拉取代码

**1.添加凭据**

 Manage Credentials -> Jenkins -> 全局凭据 -> 添加凭据，填入git仓库的账号和密码即可

![image-20220401214254107](http://oss.jankinwu.com/img/image-20220401214254107.png)

![image-20220401214300339](http://oss.jankinwu.com/img/image-20220401214300339.png)

![image-20220401214352286](http://oss.jankinwu.com/img/image-20220401214352286.png)

![image-20220401214415231](http://oss.jankinwu.com/img/image-20220401214415231.png)

![image-20220403152341524](http://oss.jankinwu.com/img/image-20220403152341524.png)

**2. 测试凭证是否可用**

回到主页，新建任务 -> 构建一个自由风格的的软件项目（FreeStyle Project）-> 输入任务名称 -> 确定

![image-20220403154202084](http://oss.jankinwu.com/img/image-20220403154202084.png)

在源码管理中选择`Git`，填入仓库的http连接，并选择刚才添加的凭据，点击保存。

![image-20220403154544885](http://oss.jankinwu.com/img/image-20220403154544885.png)

选择立即构建（Build Now）。

![image-20220403154919218](http://oss.jankinwu.com/img/image-20220403154919218.png)

点击控制台输出，可以看到项目保存位置。

![image-20220403180349602](http://oss.jankinwu.com/img/image-20220403180349602.png)

![image-20220403155343422](http://oss.jankinwu.com/img/image-20220403155343422.png)

#### 使用 Git SSH 密钥拉取代码

**1. 生成SSH密钥**

```sh
# 使用docker部署的需使用“docker exec -it jenkins bin/bash”命令进入容器内生成密钥
# 输入以下命令后，连着按三次回车键，生成秘钥
ssh-keygen -t rsa
# 获取公钥
cat ~/.ssh/id_rsa.pub
```

**2. 将SSH公钥添加到远程仓库中**

![image-20220403162252905](http://oss.jankinwu.com/img/image-20220403162252905.png)

**3. 在Jenkins中添加凭据，配置私钥**

在凭据中选择`SSH Username with private key`，填写用户名和私钥。

```sh
# 获取私钥
cat ~/.ssh/id_rsa
```

![image-20220403163545759](http://oss.jankinwu.com/img/image-20220403163545759.png)

![image-20220403163336059](http://oss.jankinwu.com/img/image-20220403163336059.png)

**4. 测试凭据是否可用**

回到主页，新建任务 -> 构建一个自由风格的的软件项目（FreeStyle Project）-> 输入任务名称 -> 确定

![image-20220403164004273](http://oss.jankinwu.com/img/image-20220403164004273.png)

在源码管理中选择`Git`，填入仓库的ssh连接，并选择刚才添加的凭据，点击保存。

![image-20220403175557825](http://oss.jankinwu.com/img/image-20220403175557825.png)

选择立即构建（Build Now）。

![image-20220403154919218](http://oss.jankinwu.com/img/image-20220403154919218.png)

点击控制台输出，可以看到项目保存位置。

![image-20220403180349602](http://oss.jankinwu.com/img/image-20220403180349602.png)

![image-20220403180653525](http://oss.jankinwu.com/img/image-20220403180653525.png)

### Maven安装和配置

使用linux安装jenkins的，参考前面使用docker安装jenkins时安装maven的步骤。环境变量则改为添加以下内容：

```sh
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
export MAVEN_HOME=/opt/maven
export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin
```

**1. 全局工具配置关联JDK、Git和Maven**

docker版自带jdk和git，可进入容器通过以下命令获取jdk和git路径，使用linux安装需提前安装好git和jdk

```sh
# 进入jenkins容器
docker exec -it jenkins /bin/bash
# 查看jdk版本
java -version
# 查看jdk路径
which java
# 查看git版本
git --version
# 查看git路径
which git
```

![image-20220403183750691](http://oss.jankinwu.com/img/image-20220403183750691.png)

![image-20220403191751463](http://oss.jankinwu.com/img/image-20220403191751463.png)

![image-20220403191813537](http://oss.jankinwu.com/img/image-20220403191813537.png)

![image-20220403191829071](http://oss.jankinwu.com/img/image-20220403191829071.png)

此处不建议直接使用自动安装maven，自动安装的maven会在首次使用maven构建项目时才下载下来，加上使用国外镜像仓库，会导致构建项目时间过长。

![image-20220404002443610](http://oss.jankinwu.com/img/image-20220404002443610.png)

**2.安装`Maven Integration`插件**

![image-20220403220802237](http://oss.jankinwu.com/img/image-20220403220802237.png)

**3. 配置环境变量**

​		配置完全局配置后去配置 jenkins 的环境变量 不然jenkins 运行打包命令会找不到 JAVA_HOME 和mvn 命令（yum安装jenkins需要配置环境变量，war包安装方式不用配置，docker 安装的jenkins属于用war包安装的，所以可以不用配置）

​		Manage Jenkins->Configure System->Global Properties ，添加三个全局变量：JAVA_HOME、M2_HOME、PATH+EXTRA

![image-20220404133825428](http://oss.jankinwu.com/img/image-20220404133825428.png)

**4. 修改maven仓库地址，方便查看**

​		docker 挂载的maven默认仓库地址为jenkins挂载目录中`~/jenkins_home/.m2/repository`下，linux 安装maven默认仓库目录则为`/root/.m2/repository`，如果不在意的话可以不改

```sh
# 创建本地仓库目录，docker安装的需要进入容器内部创建
mkdir /root/repo
# 进入配置文件
vi /opt/maven/conf/settings.xml
# 修改<localRepository>中的内容
<localRepository>/root/repo</localRepository
```

**5. 测试Maven是否配置成功**

​		构建一个maven项目。

![image-20220404143532323](http://oss.jankinwu.com/img/image-20220404143532323.png)

​		配置源码地址。

![image-20220404143917613](http://oss.jankinwu.com/img/image-20220404143917613.png)

​		填入`clean package`，保存后进行构建，如果可以编译打包成功，则说明Maven配置成功。

![image-20220404143949709](http://oss.jankinwu.com/img/image-20220404143949709.png)

![image-20220404144429762](http://oss.jankinwu.com/img/image-20220404144429762.png)

### Tomcat 安装和配置

#### 使用 docker 安装

```sh
# 首先启动一个tomcat容器，记得提前打开8080端口或关闭防火前
docker run -d -p 8080:8080 --name=tomcat tomcat:9.0
# 拷贝配置文件到宿主机
docker cp tomcat:/usr/local/tomcat /usr/local/docker/
# 可以修改tomcat配置文件（server.xml），自定义端口等
# 将webapps.dist中的文件复制到webapps目录下
cp /usr/local/docker/tomcat/webapps.dist/ /usr/local/docker/tomcat/webapps/
# 删除之前创建的tomcat
docker rm -f tomcat
# 启动正式tomcat
docker run --restart=always --name=tomcat -p 8080:8080 \
-v /usr/local/docker/tomcat:/usr/local/tomcat \
-d tomcat:9.0
```

​		在浏览器的地址栏输入：

​		http://192.168.159.100:8080/（服务器的ip和tomcat默认端口8080）

​		如果出现以下界面，则说明安装成功

![image-20220404154852237](http://oss.jankinwu.com/img/image-20220404154852237.png)

#### 配置 Tomcat

**1. 配置Tomcat用户角色权限**

​		默认情况下Tomcat是没有配置用户角色权限的，后续Jenkins部署项目到Tomcat服务器，需要用到Tomcat的用户，所以修改tomcat以下配置，添加用户及权限。

```sh
# 修改tomcat配置文件，允许jenkins访问
vi /usr/local/docker/tomcat/conf/tomcat-users.xml
# 添加以下配置，管理员密码为:tomcat/tomcat
<tomcat-users>
 <role rolename="tomcat"/>
 <role rolename="role1"/>
 <role rolename="manager-script"/>
 <role rolename="manager-gui"/>
 <role rolename="manager-status"/>
 <role rolename="manager-jmx"/>
 <role rolename="admin-gui"/>
 <role rolename="admin-script"/>
 <user username="tomcat" password="tomcat" roles="manager-gui,manager-script,tomcat,manager-status,manager-jmx,admin-gui,admin-script"/>
</tomcat-users>
# 为了能够刚才配置的用户登录到Tomcat，让所以ip都可以访问tomcat，还需要修改以下配置
vi /usr/local/docker/tomcat/webapps/manager/META-INF/context.xml
# 注释这段配置
<!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
# 重启Tomcat
docker restart tomcat
```

**2. 添加Tomcat用户凭证**

![image-20220404165729687](http://oss.jankinwu.com/img/image-20220404165729687.png)

#### 把war包部署到 tomcat 中

**1. 安装Deploy to container插件**

​		Jenkins本身无法实现远程部署到Tomcat的功能，需要安装Deploy to container插件实现。

![image-20220404160036001](http://oss.jankinwu.com/img/image-20220404160036001.png)

**2. 添加构建后操作**

  在刚才的maven项目配置中增加构建后操作步骤，应用并保存，构建成功后访问项目地址：格式为http://ip:port/context_path

![image-20220404165045833](http://oss.jankinwu.com/img/image-20220404165045833.png)

![img](http://oss.jankinwu.com/img/2347845-20210708131124441-2022501151.png)

## Pipeline

### Pipeline 简介

**1. 概念**

​		Pipeline，简单来说，就是一套运行在 Jenkins 上的工作流框架，将原来独立运行于单个或者多个节点

的任务连接起来，实现单个任务难以完成的复杂流程编排和可视化的工作。

**2. 使用 Pipeline 有以下好处**

+ 代码：Pipeline以代码的形式实现，通常被检入源代码控制，使团队能够编辑，审查和迭代其传送流程。 

+ 持久：无论是计划内的还是计划外的服务器重启，Pipeline都是可恢复的。
+ 可停止：Pipeline可接收交互式输入，以确定是否继续执行Pipeline。
+ 多功能：Pipeline支持现实世界中复杂的持续交付要求。它支持fork/join、循环执行，并行执行任务的功能。
+ 可扩展：Pipeline插件支持其DSL的自定义扩展 ，以及与其他插件集成的多个选项。

**3. 如何创建 Jenkins Pipeline**

+ Pipeline 脚本是由 **Groovy** 语言实现的，但是我们没必要单独去学习 Groovy
+ Pipeline 支持两种语法：**Declarative**(声明式)和 **Scripted Pipeline**(脚本式)语法
+ Pipeline 也有两种创建方法：可以直接在 Jenkins 的 Web UI 界面中输入脚本；也可以通过创建一个 Jenkinsfile 脚本文件放入项源码库中（一般我们都推荐在 Jenkins 中直接从源代码控制(SCM)中直接载入 Jenkinsfile Pipeline 这种方法）

### Pipeline 插件安装

![](http://oss.jankinwu.com/img/image-20220409144733050.png)

### Pipeline 语法

> Declarative声明式-Pipeline

+ pipeline：代表整条流水线，包含整条流水线的逻辑

+ stages：代表整个流水线的所有执行阶段。通常stages只有1个，里面包含多个stage。

+ stage：代表流水线中的某个阶段，可能出现n个。一般分为拉取代码，编译构建，部署等阶段。

+ steps：代表一个阶段内需要执行的逻辑。steps里面是shell脚本，git拉取代码，ssh远程发布等任意内

  容。

+ agent：指定流水线的执行位置（Jenkins agent）。流水线中的每个每段都必须在某个地方（物理机、虚拟机或Docker 容器）执行。agent部分即指定具体在哪里执行。

以上的每一项都是必需的，少任何一个，Jenkins 都会报错。



编写一个简单声明式Pipeline：

```json
pipeline { 
    agent any
    stages { 
        stage('Build') {
            steps { 
                echo 'Building..' 
            } 
        } 
        stage('Test') { 
            steps { 
                echo 'Testing..' 
            } 
        } 
        stage('Deploy') { 
            steps { 
                echo 'Deploying....' 
            } 
        } 
    }
}
```

> Scripted Pipeline脚本式-Pipeline

+ Node：节点，一个 Node 就是一个 Jenkins 节点，Master 或者 Agent，是执行 Step 的具体运行环境，后续讲到Jenkins的Master-Slave架构的时候用到。

+ Stage：阶段，一个 Pipeline 可以划分为若干个 Stage，每个 Stage 代表一组操作，比如：Build、Test、Deploy，Stage 是一个逻辑分组的概念。

+ Step：步骤，Step 是最基本的操作单元，可以是打印一句话，也可以是构建一个 Docker 镜像，由各类 Jenkins 插件提供，比如命令：sh ‘make’，就相当于我们平时 shell 终端中执行 make 命令一样。

编写一个简单脚本式Pipeline：

```json
node {
    def mvnHome
    stage('Preparation') { // for display purposes
        // Get some code from a GitHub repository
        git 'https://github.com/jglick/simple-maven-project-with-tests.git'
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.
        mvnHome = tool 'M3'
    }
    stage('Build') {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
            } else {
                bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
            }
        }
    }
    stage('Results') {
        junit '**/target/surefire-reports/TEST-*.xml'
        archiveArtifacts 'target/*.jar'
    }
}
```

### 创建 Pipeline 任务

![image-20220409163426746](http://oss.jankinwu.com/img/image-20220409163426746.png)

![image-20220409163459687](http://oss.jankinwu.com/img/image-20220409163459687.png)

通过片段生成器拉取代码：

![image-20220409165459982](http://oss.jankinwu.com/img/image-20220409165459982.png)

![image-20220409165609566](http://oss.jankinwu.com/img/image-20220409165609566.png)

将生成的脚本复制到`pull code`步骤中：

![image-20220409172113621](http://oss.jankinwu.com/img/image-20220409172113621.png)

使用 shell 脚本构建项目：

![image-20220409170904575](http://oss.jankinwu.com/img/image-20220409170904575.png)

![image-20220409172155943](http://oss.jankinwu.com/img/image-20220409172155943.png)

生成部署脚本，应用并保存：

![image-20220409171451936](http://oss.jankinwu.com/img/image-20220409171451936.png)



![image-20220409172501495](http://oss.jankinwu.com/img/image-20220409172501495.png)

构建项目并查看构建结果：

![image-20220409172706557](http://oss.jankinwu.com/img/image-20220409172706557.png)

### Pipeline Script from SCM

​		刚才我们都是直接在Jenkins的UI界面编写Pipeline代码，这样不方便脚本维护，建议把Pipeline脚本放

在项目中（一起进行版本控制）。

**1. 在项目根目录建立Jenkinsfile文件，把内容复制到该文件中，并将文件推送到远程仓库**

![image-20220416143141513](http://oss.jankinwu.com/img/image-20220416143141513.png)

**2. 在 Jenkins 中配置该脚本**

![image-20220416151554348](http://oss.jankinwu.com/img/image-20220416151554348.png)

## 触发器

Jenkins内置4种构建触发器：

+ 触发远程构建

+ 其他工程构建后触发（Build after other projects are build）

+ 定时构建（Build periodically）

+ 轮询SCM（Poll SCM）

### Git hook自动触发构建

​		轮询SCM可以实现Gitlab代码更新，项目自动构建，但是该方案的性能不佳。那有没有更好的方案呢？有的。就是利用webhook实现代码push到仓库，立即触发项目自动构建。

![img](http://oss.jankinwu.com/img/20210306010735414.png)

**1. 安装webhook插件**

​		在Jenkins上安装webhook插件，使用Github就安装Github插件，使用Gitee就安装Gitee插件，以此类推，也可以安装Generic Webhook Trigger这种通用的webhook插件。

![image-20220416165850389](http://oss.jankinwu.com/img/image-20220416165850389.png)

​		下面以Gitee为例，安装Gitee插件。

![image-20220416175214869](http://oss.jankinwu.com/img/image-20220416175214869.png)

**2. Jenkins 构建触发器**

​		在项目配置中构建触发器中选择`Gitee webhook`，复制 Jenkins 生成的URL和密码，应用并保存。

​		注意：使用Github，Gitee这些公有仓库时，需要jenkins服务器有公网IP，这样Jenkins才能接收到仓库发送的请求。

![image-20220416183106512](http://oss.jankinwu.com/img/image-20220416183106512.png)



![image-20220416183456344](http://oss.jankinwu.com/img/image-20220416183456344.png)

**2. 在Gitee中填入URL和密码**

进入源码管理配置中设置的Gitee项目中，进入 管理 -> WebHooks，添加 WebHook，填写上一步复制的URL和密码，不设密码可以不填密码，勾选 PUSH， Pull Request。

![image-20220416184339095](http://oss.jankinwu.com/img/image-20220416184339095.png)

![image-20220416184543978](http://oss.jankinwu.com/img/image-20220416184543978.png)

**3.  测试推送触发构建**

Gitee项目页面编辑一个文件提交，观察 Jenkins 任务的构建状态。

