## 访问镜像流程
![访问镜像流程](http://oss.jankinwu.com/img/Snipaste_2021-06-13_20-31-59.png)

## 容器命令

### 下载镜像

```shell
docker pull centos
```

### 新建容器并启动

```shell
docker run image # 启动容器
docker run [options] image [command]
options:
	--name=string  # 为容器指定一个名称
	-d, --detach # 后台方式运行
	-i, --interactive # 以交互模式运行容器，通常与 -t 同时使用
	-t, --tty # 启动容器后，为容器分配一个命令行，通常与 -i 同时使用
	-it # 以交互模式运行容器，进入容器查看内容（以上两条结合用法）
	-p, --publish list # 指定容器的端口
		-p ip:主机端口:容器端口
		-p 主机端口:容器端口 # 例：-p 8080:8080 （常用）
		-p 容器端口
		容器端口
	-P, --publish-all 随机指定端口
# 测试，启动并进入容器
[root@iZ2ze60her827rrgiwvu3pZ ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
mysql         5.7       2c9028880e58   4 weeks ago    447MB
hello-world   latest    d1165f221234   3 months ago   13.3kB
centos        latest    300e315adb2f   6 months ago   209MB
[root@iZ2ze60her827rrgiwvu3pZ ~]# docker run -it centos /bin/bash
[root@629965fd3499 /]# ls # 查看容器内的centos，基础版本，很多命令都是不完善的！
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

```



### 列出所有运行的容器

```shell
# docker ps 命令
docker ps # 列出当前正在运行的容器
docker ps [options]	
options:
    -a, --all  # 列出当前正在运行的容器+历史运行过的容器
    -n=int, --last int # 显示最近创建的容器
    -q, --quiet # 只显示容器的编号
[root@iZ2ze60her827rrgiwvu3pZ ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@iZ2ze60her827rrgiwvu3pZ ~]# docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED        STATUS                    PORTS     NAMES
b0f42221daff   hello-world   "/hello"   22 hours ago   Exited (0) 22 hours ago             sharp_wilson
[root@iZ2ze60her827rrgiwvu3pZ ~]# docker ps -n=1
CONTAINER ID   IMAGE         COMMAND    CREATED        STATUS                    PORTS     NAMES
b0f42221daff   hello-world   "/hello"   22 hours ago   Exited (0) 22 hours ago             sharp_wilson

```



### 退出容器

```shell
exit # 直接容器停止并退出
ctrl + P + Q # 容器不停止退出（要在大写模式下）
```

### 删除容器

```shell
docker rm 容器id # 删除指定容器，不能删除正在运行的容器，如果要强制删除 rm -f
docker rm -f $(docker ps -aq) # 删除所有容器
docker ps -a -q|xargs docker rm # 删除所有容器
```

### 启动和停止容器的操作

```shell
docker start 容器id # 启动容器
docker restart 容器id # 重启容器
docker stop 容器id # 停止当前正在运行的容器
docker kill 容器id # 强制停止当前正在运行的容器
```

### 常用其他命令

```shell
docker run -d 镜像名 # 后台启动容器（容器必须有前台应用，否则会自动停止）
docker logs [options] container # 查看日志
options:
	-f, --follow # 跟踪日志输出
	-n=int, --tail int # 查看最后n行日志
	-t, --timestamps # 显示时间戳
docker top 容器id/name # 查看容器中进程信息
docker inspect 容器id/name # 查看容器/镜像的元数据
docker exec -it 容器id/name command # 在运行的容器中执行命令
docker attach # 进入容器正在执行的终端，不会启动新的进程
```

### 从容器内拷贝文件到主机

```shell
docker ps 容器id:容器内路径 目的地主机路径
```



> 例：安装 tomcat

```shell
docker run -it --rm tomcat:9.0 # 停止运行后就会把镜像删除
```

> 例：安装 es

```shell
$ docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:tag # 当内存占用过大时，可以用 -e 设置环境变量，增加内存限制
```



### 可视化

+ portainer

  ```shell
  docker run -d -p 8088:9000 \
  --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
  ```

### commit镜像

```shell
docker commit 提交容器成为一个新的副本

# 命令和git原理类似
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```

> commit 一个tomcat

![commit 一个tomcat](http://oss.jankinwu.com/img/Snipaste_2021-06-22_00-27-39.png)

> mysql 安装

```shell
docker run -d -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --name mysql01 --restart=always mysql:5.7
```

### 具名和匿名挂载

```shell
# 匿名挂载
# docker 容器内的卷，没有指定目录的情况下都是在 “/var/lib/docker/volumes/xxx/_data”里
-v 容器内路径
docker run -d -P --name nginx -v /etc/nginx nginx

# 查看所有的 volume 的情况
docker volume ls

# 具名挂载
# -v 卷名:容器内路径
docker run -d -P --name nginx -v juming-nginx:/etc/nginx nginx

# -v /宿主内路径：容器内路径   #指定路径挂载
```

## 容器数据卷

```shell
# 启动一个父容器
docker run -it --name dc01 zzyy/centos
# dc02、dc03继承自dc01
# 容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止
docker run -it --name dc02 --volumes-from dc01 zzyy/centos
docker run -it --name dc03 --volumes-from dc01 zzyy/centos
```

## DockerFile

```shell
FROM # 基础镜像，一切从这里构建
MAINTAINER # 镜像是谁写的，姓名+邮箱
RUN # 镜像构建的时候需要运行命令
ADD # 添加内容
WORKDIR # 镜像的工作目录
VOLUME # 挂载的目录
EXPOSE # 为Docker容器设置对外的端口号
```



## 容器安装

### nginx

```shell
docker run -d --name nginx -p 8082:80 -p 8083:8083 -v /usr/local/dockerData/nginx/nginx.conf:/etc/nginx/nginx.conf -v /usr/local/dockerData/nginx/logs:/var/log/nginx -v /usr/local/dockerData/nginx/html:/usr/share/nginx/html -v /usr/local/dockerData/nginx/conf:/etc/nginx/conf.d --privileged=true nginx
```

