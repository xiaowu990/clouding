# 视频前提引导

![[Pasted image 20230826103201.png]]

> 如变量单独放到一个文件夹，tasks单独放到一个文件夹，handers也可以单独放到一个文件夹，到时候用的时候直接在playbook中引入即可

* include老旧，一般不用了

![[Pasted image 20230826105825.png]]

* 官方推荐放在/etc/ansible/roles文件夹下

![[Pasted image 20230826110100.png]]

要用到两个文件夹，一个放任务，一个放模板

剧本文件.yml和roles文件夹是平级的关系

![[Pasted image 20230826111009.png]]

![[Pasted image 20230826111235.png]]


# 目录结构
每个角色，以特定的层级目录结构进行组织
![[Pasted image 20230826120433.png]]

目录结构

- playbook.yml（调用角色）
- roles/ （==**官方推荐放在/etc/ansible/roles文件夹下**==）
    - role_name / （角色名称）
        - tasks/ （定义task,role的基本元素，至少应该包含一个名为main.yml的文件；其它的文件需要在此文件中通过include进行包含）
        - files/ （存放由copy或script等模块调用的文件）
        - vars/ （定义变量，至少应该包含一个名为main.yml的文件；其它的文件需要在此文件中通过include进行包含）
        - templates/ （template模块会自动在此目录中寻找Jinja2模板文件）
        - handlers/ （至少应该包含一个名为main.yml的文件；其它的文件需要在此文件中通过include进行包含）
        - default/（设定默认变量时使用此目录中的main.yml文件）
        - meta/（定义当前角色的特殊设定及其依赖关系,至少应该包含一个名为main.yml的文件，其它文件需在此文件中通过include进行包含）
    -
# 实例
通过roles进行安装nginx，httpd，mysql，redis
## 创建roles目录以及文件夹
```shell
[root@linux101 opt]# mkdir roles/{ngnix,httpd,mysql,redis}/tasks -pv
...
[root@linux101 opt]# mkdir roles/httpd/{handlers,files}
[root@linux101 opt]# tree /opt/
roles/
├── httpd
│   ├── files
│   ├── handlers
│   └── tasks
├── mysql
│   └── tasks
├── nginx
│   └── tasks
└── redis
    └── tasks
```

## 实现nginx的role执行逻辑
安装之前先查看是否已经安装过了nginx，如果有则卸载
```shell
[root@linux101 opt]# ansible all -m shell -a 'yum -y remove nginx'
# 查看用户和用户组是否已创建，创建则需要删除
[root@linux101 opt]# ansible all -m shell -a 'getent passwd nginx'
...
[root@linux101 opt]# ansible all -m shell -a 'getent group nginx'
...
# 如果存在则删除创建的账号
[root@linux101 opt]# ansible all -m shell -a 'userdel -r nginx'
```

在/opt/roles/nginx文件夹下创建tasks和templates文件夹

```shell
[root@linux101 nginx]# pwd
/opt/roles/nginx
[root@linux101 nginx]# ll
总用量 8
drwxr-xr-x. 2 root root 4096 5月  22 11:45 tasks
drwxr-xr-x. 2 root root 4096 5月  22 11:55 templates
```

### 1-创建nginx用户组task
在/opt/roles/nginx/tasks下创建group.yml文件，用于创建用户组，只需要填写task的name和指令即可

```yaml
[root@linux101 tasks]# cat group.yml 
- name: create group
  group: name=nginx gid=80
```

### 2-创建nignx用户task
在/opt/roles/nginx/tasks下创建group.yml文件，用于创建用户，只需要填写task的name和指令即可 一般nginx的账户是系统账户，使用相同的uid以及gid，防止在不同的服务器创建user时使用不同的uid

```shell
[root@linux101 tasks]# cat user.yml 
- name: create user
  user: name=nginx uid=80 group=nginx system=yes shell=/sbin/nologin
```

### 3-创建安装nginx的task
在/opt/roles/nginx/tasks下创建yum.yml文件

```yaml
[root@linux101 tasks]# cat yum.yml 
- name: install package
  yum: name=nginx
```

### 4-创建启动nginx的task

在/opt/roles/nginx/tasks下创建start.yml文件

```shell
[root@linux101 tasks]# cat start.yml 
- name: start service
  service: name=nginx state=started enabled=yes
```

在/opt/roles/nginx/tasks下创建restart.yml文件，用于重启服务

```yaml
[root@linux101 tasks]# cat restart.yml 
- name: start service
  service: name=nginx state=restarted
```

### 5-添加nginx的templates文件

拷贝nginx.conf文件作为j2文件到templates中

```shell
[root@linux101 nginx]# mv /etc/nginx/nginx.conf /opt/roles/nginx/templates/nginx.conf.j2
```

修改nginx.conf.j2文件

```sehll
[root@linux101 templates]# cat nginx.conf.j2 
...
user nginx;
worker_processes {{ ansible_processor_vcpus+2  }};
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;
...
```


### 6-创建调用nginx的template文件的task
在/opt/roles/nginx/tasks下创建templ.yml文件

```sehll
[root@linux101 tasks]# cat templ.yml 
- name: copy conf
  template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
```

### 7-创建 task的main.yml文件
用于组织task执行的顺序，创建/opt/roles/nginx/tasks/main.yml文件 使用`include` 关键字进行调用，书写顺序就是执行顺序

```yaml
[root@linux101 tasks]# cat main.yml 
- include: group.yml
- include: user.yml
- include: yum.yml
- include: templ.yml
- include: start.yml
```


### 8-创建playbook执行yml文件
要求和roles文件同级目录下，创建/opt/nginx_role.yml文件

```shell
- hosts: app
  remote_user: root
  
  roles:
    - role: nginx
```


### 9-执行playbook
```shell
[root@linux101 opt]# ansible-playbook nginx_role.yml
```

再次查看目录结构
```shell
[root@linux101 opt]# tree /opt/
nginx_role.yml
roles/
├── httpd
│   ├── files
│   ├── handlers
│   └── tasks
├── mysql
│   └── tasks
├── nginx
│   ├── tasks
│   │   ├── group.yml
│   │   ├── main.yml
│   │   ├── restart.yml
│   │   ├── start.yml
│   │   ├── templ.yml
│   │   ├── user.yml
│   │   └── yum.yml
│   └── templates
│       └── nginx.conf.j2
└── redis
    └── tasks
```

