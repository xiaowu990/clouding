# 镜像原理
## 联合文件系统（UnionFS）
* Union文件系统（UnionFs）是一种分层、轻量级并且高性能的文件系统，他支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下.Union文件系统是 Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像
* 特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。
## 分层原理

Linux 操作系统中的文件管理系统：
Linux 文件系统主要由boofts 和 rootfs两部分组成，其中
* bootfs：包含 bootloader（引导加载程序）和 kernel（内核）
* rootfs：root 文件系统，包含的 Linux系统中的 /dev、/proc、/bin、/etc 等标准目录和文件。
   所以，不同的Linux发行版，bootfs基本是一样的，只是 rootfs 不同，如ubuntu、centos等，相当于他们共用一个 bootfs 系统。

![[Pasted image 20230804185730.png]]
![[image-20200611165234826.png]]

* bootfs：Bootfs全名boot-file system，即引导文件系统；主要包含bootloader（系统加载）和kernel（内核）；bootloader主要用于引导加载kernel，kernel内核主要是宿主机提供的linux内核。Linux刚启动时会加载bootfs文件系统，也就是加载宿主机的linux内核。
* rootfs: 由Rootfs负责docker获取基础镜像；进行完第二步，我们就获取到了基础的linux发行版（比如是centos还是ubuntu等）
* 依赖层：此容器是用来运行什么的，我们就得安装什么依赖了。比如我们的镜像是用来运行tomcat的，而tomcat想要运行，就必须得先有jdk环境（java环境），即第三层安装jdk，安装完jdk后才能去第四层安装tomcat
* 容器层：==Docker 镜像都是只读的，当容器启动时，一个新的可写层加载到镜像的顶部，
  这一层就是我们通常说的容器层，容器之下的都叫镜像层！==
## Docker为什么要进行镜像分层
### 分层现象解释
![[Pasted image 20230804191329.png]]
* 第一张图：是nginx镜像的第一层：最下面的是bootfs，用于加载内核Kernel（如果你是在Windows上跑的docker，那这里内核就是Windows内核）；在bootfs之上的是基础镜像Debian，该基础镜像使用的是rootfs文件系统；
* 第二张图：在第一层的基础上又添加了一层，用来安装所需要的依赖镜像，比如`add emacs`；
* 第三张图：在第二层的基础上又添加了一层，用来安装具体要运行的软件，比如Apache软件；
>docker镜像是在指定了基础的发行版镜像层之后，再逐层的进行添加，比如安装软件的层，配置软件的层等，最后构建出来一个完整的镜像；在后续我们使用Dockerfile自定义镜像时，我们的每一个指令都会单独的构建出一个docker层。
### 选择分层的原因
镜像分层的一大好处就是共享资源，例如有多个镜像都来自于同一个base镜像，那么docker host只需要存储一份base镜像即可

假设我们现在有3个容器，它们都需要基于centos镜像，如果我们没有分层的结构，那么这三个容器，都必须要包含一个独立的centos镜像。如果有分层的结构，一个基础镜像层可以给三个容器去使用，即内存里只需要加载一份centos镜像，只占用100M空间；如下图
![[Pasted image 20230804191758.png]]

另外，即使多个容器共享一个base镜像，某个容器修改了base镜像的内容，例如修改了/etc下配置文件，其他容器下的/etc/下的内容是不会被修改的，修改动作只限制在单个容器内，这就是容器的写入时复制特性（Copy-on-write）

## 容器层详解
![[Pasted image 20230804192049.png]]


* 当基于一个镜像启动了一个容器后，一个新的可写层被加载到镜像的顶部，这一层被称为`容器层`，`容器层`下的层都称为镜像层。
* 当基于一个镜像启动了一个容器后，一个新的可写层被加载到镜像的顶部，这一层被称为`容器层`，`容器层`下的层都称为镜像层。


![[Pasted image 20230804192144.png]]

从表中可以看出，==只有当需要修改某一个文件时，才会将该文件从镜像层**复制**一份到容器层进行修改==，所以我们修改的其实是容器层从镜像层复制上来的备份文件，真正的系统层的该文件是不会被修改的；这种特性被称作写时复制特性（Copy-on-Write）。可见，==容器层保存的是镜像变化的部分，不会对镜像层本身进行任何修改。==当然，如果我们要删除一个镜像层已有的文件时，真正镜像层的该文件也不会被删除，只是在容器层记录一下对该文件的删除操作。

![[Pasted image 20230804192955.png]]
## 总结
Linux文件系统由bootfs 和rootfs 两部分组成

* bootfs：包含bootloader（引导加载程序）和kernel（内核）
* rootfs：root文件系统，包含的就是典型的Linux 系统中的/dev、/proc、/bin等标准目录和文件
* 不同的Linux 发行版，bootfs 基本一样，而rootfs 不同，如ubuntu，CentOS等

![[Pasted image 20230803162024.png]]



* Docker 镜像是由特殊的文件系统叠加而成
* 最低端是bootfs，并使用宿主机的bootfs（docker的内核和宿主机的内核用同一个）
* 第二层是root 文件系统rootfs ，称为base iamge
* 然后再往上可以叠加其他的镜像文件
* 统一文件系统（Union File System）技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统
* 一个镜像可以放在另一个镜像的上面。位于下面的镜像称为父镜像，最底部的镜像称为基础镜像
* 当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器

![[Pasted image 20230803162505.png]]
<center>三个问题</center>
>Docker 镜像的本质是什么？
>
 是一个分层的文件系统（是由一层一层的系统文件组成）
 
> Docker 中一个CentOS 镜像为什么只有200MB，而一个CentOS 操作系统的iso 文件要几个G？
> 
 CentOS的iso镜像文件包含bootfs和rootfs，而Docker的CentOS镜像复用操作系统的bootfs，只有rootfs和其他镜像层
 
 >Docker 中一个Tomcat 镜像为什么有500MB，而一个Tomcat 安装包只有70多MB？
 >
 由于Docker中镜像是分层的，tomcat虽然只有70多MB，但他需要依赖于父镜像和基础镜像，所以整个对外暴露的tomcat镜像大小500多MB


# 镜像制作（commit）
==如果你想要保存当前容器的状态，就可以通过commit来提交，获得一个镜像，就好比我们我们使用虚拟机的快照。==


容器转换为镜像
docker commit 容器id 镜像名称:版本号

docker save -o 压缩文件名称 镜像名称:版本号

docker load -i 压缩文件名称（相当于解压缩）
![[Pasted image 20230804192958.png]]


==docker commit -m="描述信息" -a="作者" 容器id 目标镜像名:[版本TAG]==
# Dockfile概念
* Dockerfile 是一个文本文件
* 包含了一条条的指令，每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像
* 对于开发人员，可以为开发团队提供一个完全一致的开发环境
* 对于测试人员，可以直接拿开发时所构建的镜像或者通过Dockerfile 文件构建一个新的镜像开始工作了
* 对于运维人员，在部署时，可以实现应用的无缝移植

官方的docker hub查看
![[Pasted image 20230804205436.png]]
![[Pasted image 20230804205455.png]]

>很多官方镜像都是基础包，很多功能没有，我们通常会自己搭建自己的镜像！

## DockerFile构建过程
构建步骤
1、 编写一个dockerfile文件

2、 docker build 构建称为一个镜像

3、 docker run运行镜像

4、 docker push发布镜像（DockerHub 、阿里云仓库)


基础知识
1、每个保留关键字(指令）都是必须是大写字母

2、执行从上到下顺序

3、#表示注释

4、每一个指令都会创建提交一个新的镜像层，并提交！

> Dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写dockerfile文件，这个文件十分简单
> DockerFile：构建文件，定义了一切的步骤，源代码
> DockerImages：通过DockerFile构建生成的镜像，最终发布和运行产品。
> Docker容器：容器就是镜像运行起来提供服务。



## DockerFile关键字
```sehll
FROM				# from:基础镜像，一切从这里开始构建
MAINTAINER			# maintainer:镜像是谁写的， 姓名+邮箱
RUN					# run:镜像构建的时候需要运行的命令
ADD					# add:步骤，如添加tomcat镜像，这个tomcat压缩包！添加内容 
WORKDIR				# workdir:镜像的工作目录
VOLUME				# volume:挂载的目录，就是设置一个容器卷
EXPOSE				# expose:暴露端口配置
CMD					# cmd:指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT			# 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD				# onbuild:当构建一个被继承DockerFile这个时候就会运行onbuild的指令，触发指令
COPY				# copy:类似ADD，将我们文件拷贝到镜像中
ENV					# env:构建的时候设置环境变量！

```

|关键字|作用|备注|
|---|---|---|
|FROM|指定父镜像|指定dockerfile基于那个image构建|
|MAINTAINER|作者信息|用来标明这个dockerfile谁写的|
|LABEL|标签|用来标明dockerfile的标签 可以使用Label代替Maintainer 最终都是在docker image基本信息中可以查看|
|RUN|执行命令|执行一段命令 默认是/bin/sh 格式: RUN command 或者 RUN ["command" , "param1","param2"]|
|CMD|容器启动命令|提供启动容器时候的默认命令 和ENTRYPOINT配合使用.格式 CMD command param1 param2 或者 CMD ["command" , "param1","param2"]|
|ENTRYPOINT|入口|一般在制作一些执行就关闭的容器中会使用|
|COPY|复制文件|build的时候复制文件到image中|
|ADD|添加文件|build的时候添加文件到image中 不仅仅局限于当前build上下文 可以来源于远程服务|
|ENV|环境变量|指定build时候的环境变量 可以在启动的容器的时候 通过-e覆盖 格式ENV name=value|
|ARG|构建参数|构建参数 只在构建的时候使用的参数 如果有ENV 那么ENV的相同名字的值始终覆盖arg的参数|
|VOLUME|定义外部可以挂载的数据卷|指定build的image那些目录可以启动的时候挂载到文件系统中 启动容器的时候使用 -v 绑定 格式 VOLUME ["目录"]|
|EXPOSE|暴露端口|定义容器运行的时候监听的端口 启动容器的使用-p来绑定暴露端口 格式: EXPOSE 8080 或者 EXPOSE 8080/udp|
|WORKDIR|工作目录|指定容器内部的工作目录 如果没有创建则自动创建 如果指定/ 使用的是绝对地址 如果不是/开头那么是在上一条workdir的路径的相对路径|
|USER|指定执行用户|指定build或者启动的时候 用户 在RUN CMD ENTRYPONT执行的时候的用户|
|HEALTHCHECK|健康检查|指定监测当前容器的健康监测的命令 基本上没用 因为很多时候 应用本身有健康监测机制|
|ONBUILD|触发器|当存在ONBUILD关键字的镜像作为基础镜像的时候 当执行FROM完成之后 会执行 ONBUILD的命令 但是不影响当前镜像 用处也不怎么大|
|STOPSIGNAL|发送信号量到宿主机|该STOPSIGNAL指令设置将发送到容器的系统调用信号以退出。|
|SHELL|指定执行脚本的shell|指定RUN CMD ENTRYPOINT 执行命令的时候 使用的shell|

### CMD与ENTRYPOINT的区别
#### cmd
```shell
CMD					# 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代。
ENTRYPOINT			# 指定这个容器启动的时候要运行的命令，可以追加命令
```

编写dockerfile文件
```shell
vim dockerfile-test-cmd
FROM centos 
CMD ["ls","-a"] # 启动后执行 ls -a 命令
```
构建镜像 
==docker build -f dockerfile-test-cmd -t cmd-test:0.1 .==
>后面的“.”不能丢

运行镜像
docker run cmd-test:0.1     # 由结果可得，运行后就执行了 ls -a 命令，可知正确执行了
![[Pasted image 20230804222520.png]]

错误的追加方式：
docker run cmd-test:0.1 -l
![[Pasted image 20230804222625.png]]

正确的追加方式，是替换掉
docker run cmd-test:0.1 ls -al
![[Pasted image 20230804222835.png]]

#### ENTRYPOINT
```shell
# 编写dockerfile文件
$ vim dockerfile-test-entrypoint
FROM centos
ENTRYPOINT ["ls","-a"]

# 构建镜像
$ docker build  -f dockerfile-test-entrypoint -t cmd-test:0.1 .

# 运行镜像
$ docker run entrypoint-test:0.1
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found ...

# 我们的命令，是直接拼接在我们得ENTRYPOINT命令后面的
$ docker run entrypoint-test:0.1 -l
total 56
drwxr-xr-x   1 root root 4096 May 16 06:32 .
drwxr-xr-x   1 root root 4096 May 16 06:32 ..
-rwxr-xr-x   1 root root    0 May 16 06:32 .dockerenv
lrwxrwxrwx   1 root root    7 May 11  2019 bin -> usr/bin
drwxr-xr-x   5 root root  340 May 16 06:32 dev
drwxr-xr-x   1 root root 4096 May 16 06:32 etc
drwxr-xr-x   2 root root 4096 May 11  2019 home
lrwxrwxrwx   1 root root    7 May 11  2019 lib -> usr/lib
lrwxrwxrwx   1 root root    9 May 11  2019 lib64 -> usr/lib64 ....
```


# Dockfile案例
* 自定义CentOS7镜像。要求：
1. 默认登录路径为/usr
2. 可以使用vim

* 实现步骤：
定义父镜像：FROM centos:7
定义作者信息：MAINTAINER crisp077 &lt;www.crisp077.xyz&gt;
执行安装vim命令：RUN yum install -y vim
定义默认的工作目录：WORKDIR /usr
定义容器启动执行的命令：CMD /bin/bash


![[Pasted image 20230803192425.png]]

![[Pasted image 20230803191942.png]]
通过Dockerfile文件来构建自己的镜像文件
## 构建自己的centos
Docker Hub 中 99%的镜像都是从这个基础镜像过来的 FROM scratch，然后配置需要的软件和配置来进行构建。
![[Pasted image 20230804232703.png]]


mkdir dockerfile
vim mydockerfile-centos
编写dockfile文件
```shell
FROM centos							# 基础镜像是官方原生的centos
MAINTAINER cao<1165680007@qq.com> 	# 作者

ENV MYPATH /usr/local				# 配置环境变量的目录 
WORKDIR $MYPATH						# 将工作目录设置为 MYPATH

RUN yum -y install vim				# 给官方原生的centos 增加 vim指令
RUN yum -y install net-tools		# 给官方原生的centos 增加 ifconfig命令

EXPOSE 80							# 暴露端口号为80

CMD echo $MYPATH					# 输出下 MYPATH 路径
CMD echo "-----end----"				
CMD /bin/bash						# 启动后进入 /bin/bash
```

通过这个文件构建镜像:docker build -f 文件路径 -t 镜像名:[tag] . 
==docker build -f mydocfile -t mycentos:0.1 .==
>注意点
>镜像名和版本都是自定义的
>==最后面要加一个点==
>-f:显示指定构建镜像的 Dockerfile 文件的==文件路径==
>    在当前目录下，直接写文件名
>    不在当前目录下，要写上文件路径
>    -f其实是可选的：默认情况下 Docker 会在构建过程中查找名为 `Dockerfile` 的文件作为       构建上下文中的 Dockerfile。
>-t：指定镜像名和版本号
![[Pasted image 20230805085817.png]]

![[Pasted image 20230805091545.png]]

测试
docker run -it mycentos:0.1 # 注意带上版本号，否则每次都回去找最新版latest

![[Pasted image 20230805091728.png]]

本地进行的历史变更
![[Pasted image 20230805092009.png]]
# springboot项目部署

1. 先将springboor项目上传到Linux系统中。

![[Pasted image 20230804225233.png]]

2. 定义dockerfile
定义父镜像
定义作者信息
将jar包添加到容器
定义容器启动执行的命令
build命令构建镜像

![[Pasted image 20230804230133.png]]

![[Pasted image 20230805092301.png]]

![[Pasted image 20230805092436.png]]

![[Pasted image 20230805092446.png]]

# 发布镜像
## 发布到docker hub
```shell
$ docker login --help
Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username

$ docker login -u 你的用户名 -p 你的密码

```
push镜像
![[Pasted image 20230805093834.png]]

```shell
# 会发现push不上去，因为如果没有前缀的话默认是push到 官方的library
# 解决方法：
# 第一种 build的时候添加你的dockerhub用户名，然后在push就可以放到自己的仓库了
$ docker build -t kuangshen/mytomcat:0.1 .

# 第二种 使用docker tag #然后再次push
$ docker tag 容器id kuangshen/mytomcat:1.0 #然后再次push
$ docker push kuangshen/mytomcat:1.0
—```

## 发布到阿里云镜像服务
看官网 很详细https://cr.console.aliyun.com/repository/
```shell
$ sudo docker login --username=zchengx registry.cn-shenzhen.aliyuncs.com
$ sudo docker tag [ImageId] registry.cn-shenzhen.aliyuncs.com/dsadxzc/cheng:[镜像版本号]

# 修改id 和 版本
sudo docker tag a5ef1f32aaae registry.cn-shenzhen.aliyuncs.com/dsadxzc/cheng:1.0
# 修改版本
$ sudo docker push registry.cn-shenzhen.aliyuncs.com/dsadxzc/cheng:[镜像版本号]
```


