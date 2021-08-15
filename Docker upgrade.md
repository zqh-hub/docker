#### Docker精髓

#### 镜像生成与数据卷

##### Commit实现生成镜像

```shell
docker commit -m="描述信息" -a="作者" 容器id 目标镜像名:[Tag]

[root@localhost ~]# docker run -it tomcat    启动tomcat,发现webapps下没有项目
# 现在咱们就将源tomcat进行修改，让webapps下有项目，然后再进行镜像打包
[root@localhost ~]# docker exec -it 959fac3a6e08 /bin/bash      新终端进入tomcat
root@959fac3a6e08:/usr/local/tomcat# cp -r webapps.dist/* webapps    拷贝项目到webapps下，这就是新增的层
[root@localhost ~]# docker commit -m="new tomcat" -a="coco" 959fac3a6e08 tomcatc  # 新的镜像
sha256:43c1d60400e8807c4babbb3760ea5672f702e40244ca83e88f3f9ea45d678507
[root@localhost ~]# docker images
REPOSITORY      TAG       IMAGE ID       CREATED         SIZE
tomcatc         latest    43c1d60400e8   6 seconds ago   673MB   # 自己做的镜像，以后用它就可以了
tomcat          latest    46cfbf1293b1   13 days ago     668MB

docker run -d -p 3345:8080 --name tomcat_m 43c1d60400e8  # 直接启动新生成的
```

##### 容器数据卷

```shell
将容器的目录挂载到linux上。
容器的持久化和同步操作，容器间也可以数据共享
```

![005](/Users/coco/Documents/docker/img/005.png)

```shell
docker run -it -v 主机目录:容器目录

练习：
# 将镜像centos的/home目录与linux上的/home/centos进行关联，双向绑定
[root@localhost /]# docker run -it -v /home/centos:/home centos /bin/bash

练习：
[root@localhost /]# docker run -it -p 3306:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
-v /home/mysql/conf:/etc/mysql/conf.d    linux的/home/mysql/conf 与 /etc/mysql/conf.d绑定
-e MYSQL_ROOT_PASSWORD=123456  设置密码
可以在外部使用3306端口连接
```

![006](/Users/coco/Documents/docker/img/006.png)

##### DockerFile生成镜像

```shell
# 生成镜像的第二种方式：
step 1: 编写dockerfile文件
  FROM centos
  VOLUME ["volume01","volume02"]
  CMD echo "---- end ----"
  CMD /bin/bash

step 2: build
docker build -f dockerfile文件的路径 -t 镜像名:版本号 .
[root@localhost home]# docker build -f ./dockerfile -t m_centos:0.1 .
```

![007](/Users/coco/Documents/docker/img/007.png)

##### 数据卷容器

```shell
# 在上一节build的基础上进行
两个容器实现挂载后，即使删除其中的一个，数据也不会丢失。实际是拷贝
# 1、开启docker_01
[root@localhost home]# docker run -it --name docker_01 d68a3de04578 /bin/bash
# 2、开启docker_02,并与docker_01关联
[root@localhost home]# docker run -it --name docker_02 --volumes-from docker_01 d68a3de04578
```

![009](/Users/coco/Documents/docker/img/009.png)

#### Dockerfile

##### Dockerfile介绍

![010](/Users/coco/Documents/docker/img/010.jpeg)

```
DockerFile：构建文件，定义了一切的步骤，源代码
DockerImages：通过DockerFile构建生成的镜像，最终发布和运行的产品
Docker容器：容器就是镜像运行起来，提供服务的服务器
```

##### Dockerfile命令

![011](/Users/coco/Documents/docker/img/011.jpeg)

```shell
FROM： 基础镜像，一切从这里开始
MAINTAINER:	镜像是谁写的，姓名+邮箱
RUN： 镜像构建的时候需要运行的命令
ADD： 步骤，添加东西
WORKDI： 镜像的工作目录
VOLUME： 挂载的目录位置
EXPOSE： 暴露端口  与-p 3345:8080一样
CMD： 指定容器启动时要运行的命令。注意：只有最后一个会生效，可被替代
ENTRYPOINT： 指定容器启动时要运行的命令，可以直接追加命令，注意与CMD的区别
ONBUILD： 当构建一个被继承Dockerfile,这个时候就会运行 ONBUILD 的命令。是触发命令
COPY： 类似ADD,将我们的文件拷贝到镜像中
ENV： 构建的时候设置环境变量
```

###### 练习：自定义tomcat镜像

1、上传tomcat和jdk

![012](/Users/coco/Documents/docker/img/012.png)

2、编写Dockerfile

```shell
# 基础镜像
FROM centos
# 作者信息
MAINTAINER coco<going163@163.com>
# 将read.txt拷贝到 /root/docker/tomcat/read.txt
COPY read.txt /root/docker/tomcat/read.txt

# add命令会自动解压
ADD jdk-8u161-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.52.tar.gz /usr/local/
# 安装vim命令
RUN yum -y install vim

ENV MYPATH /usr/local
# 工作空间
WORKDIR $MYPATH
# 环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_161
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.52
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.52

ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
# 开放8080
EXPOSE 8080
# 启动镜像时，运行该命令
CMD /usr/local/apache-tomcat-9.0.52/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.52/logs/catalina.out
```

3、生成镜像

```shell
docker build -t tomcatmi .
```

4、启动镜像

```shell
docker run -d -p 9090:8080 --name tomcatmi_01 -v /root/docker/build/tomcat/object_01:/usr/local/apache-tomcat-9.0.52/webapps/object_01 -v /root/docker/build/tomcat/logs:/usr/local/apache-tomcat-9.0.52/logs tomcatmi

-d:后台运行
-p:关联端口
--name:容器名字
-v:挂载目录
	/root/.../object_01:/usr/.../webapps/object_01:将linux服务器的object_01文件夹与镜像里的/webapps/object_01同步，这样直接修改linux服务器的object_01文件夹就可以了
```

5、访问

```
http://122.51.127.167:9090/
```

#### 发布镜像

##### 发布到Docker Hub

1、注册DockerHub

```shell
youmikuang
going163@163.com
zqh139499
```

2、命令行登陆docker hub账号

```shell
docker login -u youmikuang
```

3、提交

```shell
docker push tomcatmi
# 指定版本号的方式
1、docker tag 容器ID youmikuang/tomcatmi:1.0 # 这样，镜像就会多出一个youmikuang/tomcatmi
2、docker push youmikuang/tomcatmi:1.0
```

![013](/Users/coco/Documents/docker/img/013.png)

##### 发布到腾讯云镜像

1、创建命名空间

![014](/Users/coco/Documents/docker/img/014.png)

2、创建仓库

<img src="/Users/coco/Documents/docker/img/015.png" alt="015" style="zoom:33%;" />

![016](/Users/coco/Documents/docker/img/016.png)

#### Docker网络

![017](/Users/coco/Documents/docker/img/017.png)

##### 网络介绍

![018](/Users/coco/Documents/docker/img/018.png)

###### 尝试linux服务器ping容器ip

1、获取容器的ip

![019](/Users/coco/Documents/docker/img/019.png)

![](/Users/coco/Documents/docker/img/021.png)

```
每启动一个容器，就会产生一对ip，这实现了linux服务器与容器的联通，利用的是evth-pair技术
```

2、ping

![020](/Users/coco/Documents/docker/img/020.png)

###### 尝试容器与容器ping

![022](/Users/coco/Documents/docker/img/022.png)

###### 原理图

![](/Users/coco/Documents/docker/img/023.png)

![](/Users/coco/Documents/docker/img/024.png)

##### --link实现通过名称ping（过时）

```
弊端：现在是tomcat_02 与tomcat_01实现link，02ping01没问题，但是01ping02不一定成功
```

![025](/Users/coco/Documents/docker/img/025.png)

```shell
使用link能够实现ping 的原理：
	当tomcat_02执行命令后，会在其的host文件里写入tomcat_01的ip,所以tomcat_02 ping tomcat_01是可以的
```

![](/Users/coco/Documents/docker/img/026.png)

##### 自定义网络

###### 查看所有的网络

```shellshe l
网络模式
	bridge	桥接模式（推荐）
	none	不配置网络
	host		与宿主机共享模式
	container		容器网络连通
Name：
	bridge		它就是docker0
```

![](/Users/coco/Documents/docker/img/027.png)

###### 默认--net命令

```shell
# 在启动一个容器时，虽然之前的没有制定网络，但实际默认是有的
docker run -d -p 9090:8080 --name tomcat_01 tomcat
等价于：
docker run -d -p 9090:8080 --name tomcat_01 --net bridge tomcat # bridge即使docker0
```

###### 创建网络

```shell
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

--drive：指定网络模式
--subnet：子网
--gateway：网关
```

![028](/Users/coco/Documents/docker/img/028.png)

![029](/Users/coco/Documents/docker/img/029.png)

###### 查看网络

```shell
docker network inspect 网络名
```

![030](/Users/coco/Documents/docker/img/030.png)

###### 指定自定义的网络

```shell
1、指定网络，运行两个容器
```

![031](/Users/coco/Documents/docker/img/031.png)

```shellshe l
2、容器运行后，查看网络的变化：新增IP
```

![032](/Users/coco/Documents/docker/img/032.png)

```shell
3、容器之间使用容器名进行ping
```

![](/Users/coco/Documents/docker/img/033.png)

###### 使用自定义网络的好处

```shellshe l
保证不同集群之间的安全
```

##### 网络之间的联通

![034](/Users/coco/Documents/docker/img/034.png)

###### network connect命令

```shell
docker network connect 网络名 容器名
```

![035](/Users/coco/Documents/docker/img/035.png)

```shell
连通之后，网络mynet新增容器ip
# 其实就是将容器tomcat_bridge_01容器有两个IP：docker inspect 容器id
```

![036](/Users/coco/Documents/docker/img/036.png)

![037](/Users/coco/Documents/docker/img/037.png)
