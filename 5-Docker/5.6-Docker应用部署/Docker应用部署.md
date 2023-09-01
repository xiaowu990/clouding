# 部署mysql
![[Pasted image 20230802232354.png]]
* 搜索mysq|镜像
docker search mysq1

* 拉取mysq|镜像
docker pu11 mysq1:版本号

* 创建容器，设置端口映射、目录映射
在/root目录下创建mysq1目录用于存储mysq1数据信息
mkdir /root/mysq1
cd /root/mysql

```shell
docker run -id \
-p 3307:3306 \
--name=c_mysql \
-v $PWD/conf:/etc/mysql \
-v $PWD/logs:/logs \
-v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=143456 \
mysql:latest
```

-p 3307:3306 \   ：

> 将容器的3306端口映射到宿主机的3307端口，这里宿主机IP也可以写3306


--name=c_mysql \    ：给容器命名
-v $PWD/conf:/etc/mysql \ 

>将主机当前目录下的 conf/ 挂载到容器的 /etc/mysql （conf目录为mysql的配置文件，不挂载也没问题）

-v $PWD/1ogs:/logs  \

>将主机当前目录下的 logs 目录挂载到容器的 /logs （logs目录为mysql的日志目录，不挂载也没影响）

-v $PWD/data:/var/1ib/mysq1 \

>将主机当前目录下的data目录挂载到容器的 /var/lib/mysql （data目录为mysql配置的数据文件存放路径，这个还是建议挂载，是存储数据的，容器down掉，还能再次挂载数据。）

-e MYSQL ROOT PASSWORD=143456 \

>初始化 root 用户的密码


* 进入mysql容器:docker exec -it c_mysql /bin/bash
* 登录mysql :mysql -uroot -p
# 部署tomcat
## 部署
```shell
mkdir tomcat  #创建一个目录用来专门存储tomcat信息
cd tomcat

docker run -id --name=tomcat001 \
-p 8900:8080 \
-v $PWD:/usr/local/tomcat/webapps \
tomcat 
```

>-p 8080:8080：将容器的8080端口映射到主机的8080端口
 -v $PWD:/usr/local/tomcat/webapps：将主机中当前目录挂载到容器的webapps

* Docker 命令中，==`$PWD` 通常用于引用当前工作目录==，并将其映射到 Docker 容器中的某个目录，以便在容器中使用这些文件。
## 测试
在宿主机下，的tomcat目录下，创建一个test目录，在test目录里建一个index.html文件，因为是映射到容器里的webapp中的
所以在浏览器输入==175.24.174.201:8900/test/index.html==会出现这个效果
![[Pasted image 20230804160648.png]]

![[Pasted image 20230804160305.png]]

![[Pasted image 20230804155358.png]]
# 部署Nginx
## 部署
```shell
<br class="Apple-interchange-newline"><div></div>
# 在/root目录下创建nginx目录用于存储nginx数据信息
mkdir ~/nginx
cd ~/nginx

# 在~/nginx/conf/下创建nginx.conf文件,粘贴下面内容
mkdir conf
cd conf
vim nginx.conf  #就是nginx.conf拷贝来的
```

```shell
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```

```shell
以下命令一定要回到/~/nginx目录下再执行，不然$PWD就不是当前目录了
docker run -id --name=nginx01 \
-p 80:80 \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/usr/share/nginx/html \
nginx
```

- -p 80:80：将容器的 80端口映射到宿主机的 80 端口。
- -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf：将主机当前目录下的 /conf/nginx.conf 挂载到容器的 :/etc/nginx/nginx.conf。配置目录
- -v $PWD/logs:/var/log/nginx：将主机当前目录下的 logs 目录挂载到容器的/var/log/nginx。日志目录

![[Pasted image 20230804163148.png]]
## 测试
在nginx里的html中，创建一个index页面，直接ip+端口即可访问到（防火墙开发指定的端口）

![[Pasted image 20230804163218.png]]

docker hub里面可以查看到详细的信息

![[Pasted image 20230804145027.png]]


# 部署redis
1. 搜索redis镜像

docker search redis

2. 拉取redis镜像
   
docker pull redis:5.0

3. 创建容器，设置端口映射
    
docker run -id --name=c_redis -p 6379:6379 redis:5.0

4. 使用外部机器连接redis

./redis-cli.exe -h 192.168.149.135 -p 6379
