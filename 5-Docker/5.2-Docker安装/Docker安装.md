# 准备工作
1. Linux内核要求3.0及以上：uname -r
2. centOS要求7及以上：cat /etc/os-release 

![[Pasted image 20230802095955.png]]

3. 卸载旧版本
```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  ```
![[Pasted image 20230802100124.png]]

# Docker存储库安装
1. 安装需要的包
yum install -y yum-utils
2. 设置仓库的镜像
   1. 官方
   ```shell
   yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    ```
   1. 阿里云
   ```shell
   yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    ```
3. 更新软件包索引
yum makecache fast
4. 安装docker社区版
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
5. 启动docker
systemctl start docker

6. 使用docker version 查看是否安装成功
docker version
7. 测试
docker run hello-world
8. 查看下载的hello-world镜像
docker images
9. 卸载docker
   1. 卸载 Docker Engine、CLI、containerd 和 Docker Compose 软件包
   sudo yum remove docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
   2. 主机上的映像、容器、卷或自定义配置文件不会自动删除。要删除所有映像、容器和卷
   sudo rm -rf /var/lib/docker
   sudo rm -rf /var/lib/containerd
>==/var/lib/docker 是docker的默认工作路径！==

![[Pasted image 20230802101550.png]]

# 阿里云镜像加速(第一次尝试时导致无法启动了！：问题出在添加的json文件中，没特殊要求就不要阿里云加速了)
![[Pasted image 20230802102326.png]]

```shell
#1.创建一个目录
sudo mkdir -p /etc/docker

#2.编写配置文件
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": "https://vg0ike0i.mirror.aliyuncs.com"]
}
EOF

#3.重启服务
sudo systemctl daemon-reload
sudo systemctl restart docker
```
