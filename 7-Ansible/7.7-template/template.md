# 前提了解

![[Pasted image 20230825162238.png]]

* template是模块，templates是文件夹

# 基本介绍
* template文件必须存放于templates目录下，且命名为 .j2 结尾
* 只能用于playbook,不能用于命令行
* 使用说明
   * ==template文件必须存放于templates目录下，且命名为 .j2 结尾==
   * ==yaml/yml 文件需和templates目录平级==
```shell
./
├── my-playbook.yml
└── templates
  └── xx.j2
```

# 案例-安装nginx
> [!note] 获取nginx

先安装，然后查看配置文件
yum -y install nginx

查看安装目录
rpm -ql nginx
![[Pasted image 20230825184808.png]]

安装nginx需要EPL源头，
```
yum repolist # 查看是否有EPL源，给安装nginx做准备
# 如果没有，则需要安装epel
yum install -y epel-release
```

![[Pasted image 20230825185222.png]]

> [!note] 启动Nginx服务

systemctl start nginx

查看 master process 和 worker process 进程个数

![[Pasted image 20230825185657.png]]

# 创建templates文件夹

创建templates文件夹，要求和playbook执行的yml文件在同一目录下

![[Pasted image 20230825190417.png]]

## 编辑执行的yaml文件
创建nginx-deploy.yml文件，如果nginx.conf.j2没有放在相对路径的templates目录下，那么src需要配置绝对路径

```shell

- hosts: [wbsr]
  remote_user: root

  tasks:
    - name: install package
      yum: name=nginx
    - name: copy template
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
      #如果没有放到templates目录下，这里src就要写成绝对路径了
    - name: start service
      service: name=nginx state=started enabled=yes 
      # enabled yes表示开机启动
```

执行检查playbook.yml文件命令
```
ansible-playbook -C nginx-deploy.yml
```

执行
```
ansible-playbook nginx-deploy.yml
```

## 检查安装结果
```
ansible app -m shell -a 'rpm -q nginx'
ansible app -m shell -a 'ss -ntpl' #查看端口以及进程
ansible app -m shell -a 'ps aux | grep nginx'
```

![[Pasted image 20230825194812.png]]

![[Pasted image 20230825194827.png]]

默认worker进程数是和cpu相关，4个内核,4个worker
下面是2个也是同样
# 更改操作:使worker进程是当前CPU个数的倍数
![[Pasted image 20230825195443.png]]

进入到nginx.conf.j2文件中去
![[Pasted image 20230825195621.png]]

![[Pasted image 20230825195755.png]]

![[Pasted image 20230825195857.png]]

# 改变Nginx服务的端口
设置第一个机器的nginx服务端口为81
设置第二季机器的nginx服务端口为82

在主机清单中定义变量
![[Pasted image 20230825214121.png]]

>[!warning] 补充：也可以采用另一种方式定义变量-在playbook中定义
>![[Pasted image 20230825214928.png]]


打开nginx配置文件：vim templates/nginx.conf.j2

在其中调用变量名![[Pasted image 20230825214418.png]]

![[Pasted image 20230825214654.png]]

> 这里只要执行一次就行，因为有handler了

### 详细说明
在主机清单中给不同的机器设置不同的端口号，作为nginx访问使用的端口号 编辑/etc/ansible/hosts中添加端口号
```yaml
[app]
192.168.10.102 http_port=8081
192.168.10.103 http_port=8082
192.168.10.104 http_port=8083
```

在templates/nignx.conf.j2中使用http_port变量
```yaml
user nginx;
worker_processes {{ ansible_processor_vcpus//2 }};
...
http {
   ...
    server {
        listen       {{ http_port  }}; # 配置http_port
        listen       [::]:{{ http_port  }};
        server_name  _;
        root         /usr/share/nginx/html;
   ...
    }

}
```

执行命令
```
ansible-playbook nginx-deploy.yml
```

注意：如果nginx出现了 bind() to 0.0.0.0:8082 failed (13: Permission denied)，那么需要关闭selinux，临时方式如下 如果需要永久关闭selinux，请编辑/etc/selinux/config文件，将SELINUX=disabled。之后将系统重启一下即可
```
[root@linux103 nginx]# getenforce
Enforcing
[root@linux103 nginx]# setenforce 0
[root@linux103 nginx]# getenforce
Permissive
```

查看执行结果
```
ansible app -m shell -a 'ss -ntpl'
```

如果在nginx-deploy.yml中定义变量，那么playbook的执行配置文件中优先级最高

```yaml
- hosts: app
  remote_user: root
  vars:
    - http_port: 8088

  tasks:
    - name: install package
      yum: name=nginx
    - name: copy template
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf    
      notify: restart service
    - name: start service
      service: name=nginx state=started enabled=yes
  
  handlers:
    - name: restart service
      service: name=nginx state=restarted
```

# template使用if
语法 `{% if 判断条件 %} xxx {% endif %}`

定义变量

```yaml
- hosts: app
  remote_user: root
  vars:
    nginx_vhosts:
      - listen: 8080  #列表 键值对
```


使用变量

```yaml
{% for vh in nginx_vhosts %}
server {    #重复执行server代码
    listen {{ vhost.listen | default('80 default_server') }};

    {% if vh.server_name is defined %}
    server_name {{ vhost.server_name }};
    {% endif %}

    {% if vh.root is defined %}
    root {{ vhost.root }};
    {% endif %}

}
{% endfor %}
```


# template使用for
## 基本介绍

语法 `{% for item in vars %} xxx {% endfor %}`
以for开头，以for结尾，保持对称

![[Pasted image 20230825234308.png]]



## 视频案例
### 以列表的方式
> [!attention] 创建playbook


![[Pasted image 20230825233429.png]]

==*{{ port }}变量的值来自于for循环，for循环的ports就来自于testfor.yml中定义的81,82,83*==

> [!danger] 创建template

![[Pasted image 20230825233743.png]]

> [!note] 最终的效果

最终生成的效果如下

![[Pasted image 20230825233651.png]]

> [!summary] 验证


![[Pasted image 20230825234026.png]]
### 以字典的方式（键值对）

改为键值对

![[Pasted image 20230825234647.png]]

![[Pasted image 20230825234830.png]]

验证



## 键值对

定义变量

```yaml
- hosts: app
  remote_user: root
  vars:
    nginx_vhosts:
      - listen: 8080  #列表 键值对
```

在templates/nginx.conf.j2文件中使用

```yaml
{% for vh in nginx_vhosts %}
server {    #重复执行server代码
listen {{ vh.listen | default('80 default_server') }};

...
}
{% endfor %}
```


生成结果

```yaml
server {
  listen 8080
}
```

## 列表
定义变量

```
- hosts: mageduweb
  remote_user: root
  vars:
    nginx_vhosts:
      - web1
      - web2
      - web3
  tasks:
    - name: template config
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
```

在templates/nginx.conf.j2文件中使用列表变量

```
{% for vh in nginx_vhosts %}
server {
    listen {{ vh }}
}
{% endfor %}
```

生成结果

```
server {
    listen web1
}
server {
    listen web2
}
server {
    listen web3
}
```

# 实例-使用for与if

编写playbook的执行yaml文件，app.yaml

```
- hosts: app
  remote_user: root
  vars:
    nginx_vhosts:
      - web1:
        listen: 8080
        root: '/var/www/stt/web1'
      - web2:
        listen: 8080
        server_name: web2.stt.com
        root: '/var/www/stt/web2'   
      - web3:
        listen: 8080
        server_name: web3.stt.com
        root: '/var/www/stt/web3'
        
  tasks:
    - name: template config to
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
```


编写 nginx.conf.j2文件

```
{% for vh in nginx_vhosts %}
server {
 listen {{ vh.listen }}
 {% if vh.server_name is defined %}
 server_name {{ vh.server_name }}
 {% endif %}
 root {{ vh.root }}
}
{% endfor %}
```

生成结果

```
server {
  listen 8080
  root /var/www/stt/web1/
}
server {
  listen 8080
  server_name: web2.stt.com
  root /var/www/stt/web2/
}
server {
  listen 8080
  server_name: web3.stt.com
  root /var/www/stt/web3/
}
```
