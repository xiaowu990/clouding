# 服务相关命令
* 启动docker 服务：
systemctl start docker

* 停止docker 服务：
systemctl stop docker

* 重启docker 服务：
systemctl restart docker

* 查看docker 服务状态：
systemctl status docker

* 设置开机启动docker：
systemctl enable docker
# 镜像相关命令
* 查看镜像：
   * 查看本地所有的镜像
      docker images
   * 查看所有镜像及其id
      docker images -q  

* 搜索镜像：从网络中查找需要的镜像
docker search 镜像名称

* 拉取镜像：从Docker 仓库下载镜像到本地，镜像名称格式为，如果版本号不指定则是最新的版本。如果不知道镜像版本，可以去docker hub 搜索对应镜像查看名称
docker pull 镜像名称[:版本号]
>不加版本号就是默认的下载最新版

* 删除镜像：
   * 删除本地镜像
      docker rmi 镜像id/镜像名称[:版本号] 
   * 删除本地所有镜像
      docker rmi 'docker images -q' 


![[Pasted image 20230802162959.png]]

![[Pasted image 20230802163018.png]]
# 容器相关命令

==不会就用help：docker run --help(查看run命令)==

* 查看容器
docker ps # 查看正在运行的容器
docker ps -a # 查看所有容器（包括正在运行的，已经退出的）

* 创建并启动容器
1. 格式(-t直接进入)：docker run -it --name=容器名字 镜像名字: 镜像版本号 /bin/bash
2. 格式(-d不直接进入)：docker run -id --name=容器名字 镜像名字: 镜像版本号 /bin/bash
      * 手动进入的操作语句：docker exec -it 名字 /bin/bash
      * -d也是让docker进入守护态运行，守护态进程是一个后台进程，一旦启动就会一直运行，等待用户发送命令或请求来管理 Docker 容器
>"="也可以写成空格，/bin/bash是进入容器的初始化指令

如：docker run -it --name=c1 centos:7 /bin/bash (启动容器，镜像为centos：7，直接进入容器，退出容器直接停止容器)
docker run -id --name=c2 centos:7 (没有直接进入容器，需要操作语句进入容器，退出容器不会暂停容器)，<span style="background:#fdbfff">而且这里加不加/bin/bash都不会直接进入容器。主要是-d影响的</span>

>参数说明：
>-i：保持容器运行。通常与-t同时使用。加入这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭(==加了-i，容器就算没有客户端连接，也会一直运行着，不然没有连接会关闭==)
>-t：为容器重新分配一个伪输入终端，进入容器内部会分配一个终端
 -d：以守护（后台）模式运行容器。创建一个容器在后台运行，不会自动进入，需要命令进入。==退出后容器是不会关闭的==
 -it创建的容器一般称为交互式容器；创建的容器一般称为守护式容器-id
 >**-p:**（小写的p) 指定端口映射，格式为：主机(宿主)端口:容器端口
 >**-P:**（大写的P） ：随机指定端口
 

* 退出容器：exit

* 进入容器 docker exec 参数  

* 停止容器 docker stop 容器名称

* 强制停止容器 docker kill 容器名或id

* 启动容器 docker start 容器名称

* 删除容器：docker rm 容器id(或者容器名)
>如果容器是运行状态则删除失败，需要停止容器才能删除

* 删除所有容器：docker rm 'docker ps -aq'

* 查看容器信息  docker inspect 容器名称(==要退出容器再查找才行==)


![[Pasted image 20230802184746.png]]

![[Pasted image 20230802185149.png]]

![[Pasted image 20230802190203.png]]

![[Pasted image 20230802191731.png]]