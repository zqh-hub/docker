#### Docker

##### Docker基本组成

![001](/Users/eric/Documents/data/docker/img/001.jpeg)

##### Docker安装

```shell
1、卸载旧版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
2、下载安装包
yum install -y yum-utils
3、设置镜像库
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
4、更新索引
yum makecache fast
5、安装docker引擎
yum install docker-ce docker-ce-cli containerd.io
6、启动
systemctl start docker
# Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details. -----> 关闭防火墙
7、查看是否成功
docker version
8、运行hello world
docker run hello-world
# 存在 “Hello from Docker! ”既是成功
9、查看镜像
docker images
```

##### hello-world运行流程

![002](/Users/eric/Documents/data/docker/img/002.png)

##### 底层原理

###### docker是怎样工作的

```python
Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上，通过socket从客户端访问
DockerServer接收到Docker-Client的指令，就会执行命令
```

![003](/Users/eric/Documents/data/docker/img/003.png)

###### Docker为什么比VM快

![004](/Users/eric/Documents/data/docker/img/004.jpeg)

```
1、Docker有着比VM更少的抽象层
2、Docker利用的是宿主机的内核，VM需要是Guest OS
所以说，新建一个容器的时候，Docker不需要想虚拟机一样重新加载一个操作系统内核，避免引导。虚拟机是加载Guest OS,分钟级别，而Docker是利用宿主机的操作系统，省略了这个负载的过程
```

##### Docker常用命令

```
命令文档：
https://docs.docker.com/reference/
```

###### 帮助命令

```shell
docker version   # 查看docker版本信息
docker info    # 显示docker的系统信息，包括镜像和容器的数据
docker 命令 --help
```

###### 镜像命令

1、docker images ：查本地所有的镜像

```shell
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   4 months ago   13.3kB

# 解释
REPOSITORY     镜像的仓库源
TAG        镜像的标签
IMAGE ID       镜像的id
CREATED      镜像的创建时间
SIZE     镜像的大小

# 可选参数
--all , -a		显示所有的镜像         docker images -a
--filter , -f		过滤
--quiet , -q		只显示IMAGE ID            docker images -aq
```

2、docker search ：搜索镜像

```shell
[root@localhost ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11188     [OK]

# 可选参数
-f, --filter      过滤
[root@localhost ~]# docker search mysql -f=STARS=3000    过滤出STARS > 3000的
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   11188     [OK]       
mariadb   MariaDB Server is a high performing open sou…   4251      [OK] 
```

3、docker pull 下载镜像

```shell
[root@localhost ~]# docker pull mysql
Using default tag: latest       # 没有指定版本，就下载最新版本
latest: Pulling from library/mysql
33847f680f63: Pull complete     # 分层下载，好处：如果需要更新，只需要更新需要更新的层就行了
5cb67864e624: Pull complete 
1a2b594783f5: Pull complete 
... ...
Digest: sha256:8b928a5117cf5c2238c7a09cd28c2e801ac98f91c3f8203a8938ae51f14700fd  # 签名
Status: Downloaded newer image for mysql:latest 
docker.io/library/mysql:latest # 真实地址 
# docker pull docker.io/library/mysql 等价于 docker pull mysql

[root@localhost ~]# docker pull mysql:5.7   指定版本
```

5、docker rmi

```shell
[root@localhost ~]# docker rmi -f 8cf625070931   # 指定容器id删除
[root@localhost ~]# docker rmi -f 8cf625070931  8cf625070931  # 删除多个容器
[root@localhost ~]# docker rmi -f $(docker images -aq)  # 删除所有的容器
```

###### 容器命令

```shell
1、下载容器
docker pull centos
2、启动容器
docker run [可选参数] image

# 可选参数
--name="xx"   指定容器名字
-d				后台方式运行
-it				使用交互方式运行，进入容器查看内容
-p				指定容器的端口
		-p ip:主机端口：容器端口
		-p 主机端口：容器端口（常用）
		-p 容器端口
		容器端口
-P				随机指定端口
# 运行容器
[root@localhost /]# docker run -it centos /bin/bash
[root@fa9877ddf76b /]# ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr

# 退出容器
[root@fa9877ddf76b /]# exit 
exit
```

###### 查看运行的容器

```shell
docker ps 命令
-a         列出当前与历史运行过的容器
-n=?       显示几个
-q				 只显示容器的编号
# 正在运行的容器
[root@localhost /]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# 曾经运行的容器
[root@localhost /]# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS                     PORTS 
fa9877ddf76b   centos         "/bin/bash"   3 minutes ago   Exited (0) 2 minutes ago
8eb67be0c71b   centos         "/bin/bash"   6 minutes ago   Exited (0) 4 minutes ago            
f3f9aacae8c4   d1165f221234   "/hello"      2 days ago      Exited (0) 2 days ago

# 显示所有的容器的id
[root@localhost /]# docker ps -qa
fa9877ddf76b
8eb67be0c71b
f3f9aacae8c4
```

###### 退出容器

```shell
1、exit      容器会停止
2、Ctrl + P + Q				容器不会停止
```

删除容器

```shell
docker rm 容器id   # 删除指定的容器，不能删除正在运行的容器
docker rm -f $(docker ps -aq)   # 删除所有的容器

docker ps -a -q | xarge docker rm			# 删除所有容器
```

启动和停止容器

```shell
docker start 容器id       # 开始
docker restart 容器id     # 重启
docker stop 容器id        # 停止当前正在运行的容器
docker kill 容器id        # 强制停止当前容器

[root@localhost /]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
c1327849a083   centos    "/bin/bash"   About a minute ago   Up About a minute             goofy_swanson
[root@localhost /]# docker stop c1327849a083
c1327849a083
```

#### 常用其他命令

###### 后台启动

```shell
docker run -d centos
# 注意，后台启动后，使用docker ps 查看正在运行的容器为空，是因为后台启动需要一个前台进行，当docker发现没有前台进程时，就自动停止
```

###### 查看日志命令

```shell
docker logs 参数

docker logs -tf 容器id
docker logs -tf --tail 行数 容器id
[root@localhost /]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS
e16f7c6ee9e1   centos    "/bin/bash"   39 seconds ago   Up 37 second
[root@localhost /]# docker logs -f --tail -t e16f7c6ee9e1
```

###### 查看容器中的进程信息

```shell
top

[root@localhost /]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
e16f7c6ee9e1   centos    "/bin/bash"   8 minutes ago   Up 8 minutes             blissful_bohr
[root@localhost /]# docker top e16f7c6ee9e1
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                8021                8002                0                   21:46               pts/0               00:00:00            /bin/bash
[root@localhost /]# 
```

###### 查看镜像元数据

```shell
docker inspect 容器id
```

###### 进入正在运行的容器

```shell
# 方式1:
docker exec -it 容器id bashShell   # 进入新的终端

[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED        STATUS        PORTS     NAMES
e16f7c6ee9e1   centos    "/bin/bash"   23 hours ago   Up 23 hours             blissful_bohr
[root@localhost ~]# docker exec -it e16f7c6ee9e1 /bin/bash

# 方式2:
docker attach 容器id     # 会进入正在执行的终端，不会启动新的进程
```

###### 从容器内拷贝文件到主机

```shell
docker cp 容器id:路径 主机路径

[root@e16f7c6ee9e1 home]# touch test.py
[root@e16f7c6ee9e1 home]# ls
test.py
[root@e16f7c6ee9e1 home]# read escape sequence
[root@localhost ~]# docker cp e16f7c6ee9e1:/home/test.py /home
[root@localhost ~]# cd /home/
[root@localhost home]# ls
test.py
```

###### 练习：安装Nginx

```shell
[root@localhost home]# docker pull nginx   # 下载nginx
[root@localhost home]# docker run -d --name nginx_01 -p 3344:80 nginx   # 后台运行nginx
此时就可在浏览器访问"http://111.67.204.86:3344/"了
```

![004](/Users/eric/Documents/data/docker/img/004.png)

###### 练习：安装tomcat

```shell
```

