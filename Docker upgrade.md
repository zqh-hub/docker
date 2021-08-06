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

![005](/Users/eric/Documents/data/docker/img/005.png)

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

![006](/Users/eric/Documents/data/docker/img/006.png)

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

![007](/Users/eric/Documents/data/docker/img/007.png)

##### 数据卷容器

```shell
# 在上一节build的基础上进行
两个容器实现挂载后，即使删除其中的一个，数据也不会丢失。实际是拷贝
# 1、开启docker_01
[root@localhost home]# docker run -it --name docker_01 d68a3de04578 /bin/bash
# 2、开启docker_02,并与docker_01关联
[root@localhost home]# docker run -it --name docker_02 --volumes-from docker_01 d68a3de04578
```

![009](/Users/eric/Documents/data/docker/img/009.png)

#### Dockerfile

##### Dockerfile介绍

![010](/Users/eric/Documents/data/docker/img/010.jpeg)

```
DockerFile：构建文件，定义了一切的步骤，源代码
DockerImages：通过DockerFile构建生成的镜像，最终发布和运行的产品
Docker容器：容器就是镜像运行起来，提供服务的服务器
```

##### Dockerfile命令

![011](/Users/eric/Documents/data/docker/img/011.jpeg)

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
