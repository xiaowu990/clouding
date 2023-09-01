# 命名/定义/调用
![[Pasted image 20230824213154.png]]

"variables"（变量）用于存储和管理数据，以便在Playbook的不同任务和角色之间共享和重用。变量可以包含任意类型的数据，例如字符串、数字、列表、字典等。

变量命名
仅能由字母、数字和下划线组成，且只能以字母开头


变量定义
key=value
```
http_port=80
```

变量调用
* 在Playbook中的任务、模块和模板中使用变量。
通过{{ variable_name }} 调用变量，且变量名前后建议加空格，有时用“{{ variable_name }}”才生效
```yaml
tasks:
  - name: Display variable value
    debug:
      msg: "The variable value is {{ global_var }}"

在上述示例中，任务使用`debug`模块来显示全局变量`global_var`的值。
```


# facts
## 举例-快速了解
当你需要访问远程主机的系统信息时，你可以直接使用这些变量，而不需要额外的配置或收集信息的步骤。
下面是一个示例，演示如何使用远程主机的变量：

```yaml
- name: Display hostname and operating system
  debug:
    msg: "Hostname: {{ ansible_hostname }}, OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
```

在上述示例中，`ansible_hostname`、`ansible_distribution`和`ansible_distribution_version`是远程主机的变量。通过在`debug`任务中使用这些变量，我们可以显示主机名和操作系统信息。

另外，你还可以通过`ansible_facts`变量访问所有主机的事实信息。例如，你可以使用以下任务来显示远程主机的所有变量：

```yaml
- name: Display all facts
  debug:
    var: ansible_facts
```
这将显示远程主机的所有变量和事实信息。

## 基本概念
Ansible Facts 是 Ansible 在被托管主机上自动收集的变量。它是通过在执行 Ad-Hoc 以及 Playbook 时使用 setup 模块进行收集的，并且这个操作是默认的。
因为这个收集托管主机上的 Facts 比较耗费时间，所以可以在不需要的时候关闭 setup 模块。收集的 Facts 中包含了托管主机特有的信息，这些信息可以像变量一样在 Playbook 中使用。
**收集的 Facts 中包含了以下常用的信息：**
主机名、内核版本、网卡接口、IP 地址、操作系统版本、环境变量、CPU 核数、可用内存、可用磁盘 等等……

>* 它的声明和赋值完全有Ansible 中的 setup 模块帮我们完成。
>* 它收集了有关被管理服务器的操作系统版本、服务器IP地址、主机 名，磁盘的使⽤情况、CPU个数、内存⼤⼩等等有关被管理服务器的私 有信息。 
>* 在每次PlayBook运⾏的时候都会发现在PlayBook执⾏前都会有⼀ 个Gathering Facts的过程。这个过程就是收集被管理服务器的Facts信息过程


![[Pasted image 20230814162953.png]]

手动收集/查看:ansible all -m setup | less
![[Pasted image 20230814155217.png]]

## 过滤Facts
facts 信息量很⼤，使⽤Facts 模块中的filter参数去过滤我们想要的信息。

例如几个过滤的命令如下

只查询主机名：ansible all -m setup -a 'filter="ansible_nodename"'   
只查询主机内存大小：ansible all -m setup -a 'filter="ansible_memtotal_mb"'  
只查询系统版本ansible all -m setup -a 'filter="ansible_distribution_major_version"' 
只查询主机：cpu个数ansible all -m setup -a 'filter="ansible_processor_vcpus"'  
支持通配符：ansible all -m setup -a 'filter=*address*'


或者grep

```shell
[root@linux101 opt]# ansible all -m setup | grep ansible_memtotal_mb
        "ansible_memtotal_mb": 3934, 
        "ansible_memtotal_mb": 3934, 
        "ansible_memtotal_mb": 3934, 
```


## 在PlayBook中去使⽤Facts 变量

通过setup中读取的ansible_fqdn变量值创建

```yaml
- hosts: app
  remote_user: root
  
  tasks:
    - name: create log file
      file: name=/opt/{{ ansible_fqdn }}.log state=touch mode=500 owner=stt

```

检查playbook执行文件，再执行
ansible-playbook -C app.yml
ansible-playbook app.yml

检查执行的结果
ansible app -a 'ls /opt/ -l'

## 在PlayBook中去关闭Facts 变量的获取

![[Pasted image 20230824235358.png]]


# 在主机清单中定义变量
在/etc/ansible/hosts(主机清单)中定义变量
## 普通变量
* 普通变量：主机组中单个主机单独定义的东西，其优先级高于公共变量(单个主机 )，即==设置变量的单个主机有效==

![[Pasted image 20230824213742.png]]

>以上就是对不同的主机有单一定义


![[Pasted image 20230824214134.png]]

创建app.yml，对主机清单中的变量进行调用

```yaml
- hosts: app
  remote_user: root
 
  tasks:
    - name: set hostname
      hostname: name=www{{ http_port }}.stt.com
```

执行：先检测，在执行
```yaml
ansible-playbook -C app.yml
ansible-playbook app.yml
```

检查是否修改成功
```yaml
ansible-playbook app -a 'hostname'
```

## 公共变量/组变量

![[Pasted image 20230824215324.png]]

![[Pasted image 20230824215429.png]]

![[Pasted image 20230824215655.png]]



针对主机组中所有主机定义统一变量(一组主机的同一类别)

示例：修改hosts，添加组公共变量
```shell
[app]
192.168.10.102 http_port=81 # 单个主机变量
192.168.10.103 http_port=82
192.168.10.104 http_port=83

[app:vars] # 组变量
nodename=www
domainname=stt.com
```

修改app.yml文件

```yaml
- hosts: app
  remote_user: root
 
  tasks:
    - name: set hostname
      hostname: name={{ nodename }}{{ http_port }}.{{ domainname }}
```

## 组嵌套变量
组还可以包含其它的组，也可向组中的主机指定变量

```shell
[apache]
httpd1.stt.com
httpd2.stt.com

[nginx]
ngx1.stt.com
ngx2.stt.com

[websrvs:children] # 将上面2个组进行组合
apache
nginx

[webservers:vars] # 定义嵌套组队的变量
ntp_server=ntp.stt.com
```

```shell
组嵌套
inventory中，组还可以包含其它的组，并且也可以向组中的主机指定变量。
这些变量只能在ansible-playbook中使用，而ansible命令不支持

示例：
    [apache]
    httpd1.magedu.com
    httpd2.magedu.com
    
    [nginx]
    ngx1.magedu.com
    ngx2.magedu.com
    
    [websrvs:children]
    apache
    nginx
    
    [webservers:vars]
    ntp_server=ntp.magedu.com
   
```








# playbook中定义变量

![[Pasted image 20230824212808.png]]

在playbook中使用vars关键字，定义变量
```yaml
vars:
  - var1: value1
  - var2: value2

```

示例：创建app.yml，声明2个变量并执行

```yaml
- hosts: websrvs
  remote_user: root
  vars:
    - pkname1: httpd
    - pkname2: vsftpd
 
  tasks:
    - name: install package1
      yum: name={{ pkname1 }}
    - name: install package2
      yum: name={{ pkname2 }}
```

执行命令
ansible-playbook app.yml

检查是否安装成功：查询安装的版本信息
ansible websrvs -m shell -a 'rpm -q httpd vsftpd'

卸载刚才的安装
ansible websrvs -m shell -a 'yum -y remove httpd vsftpd'




# 通过执行命令指定
![[Pasted image 20230824210513.png]]

* ansible-playbook  文件名–e 选项指定
```yaml
ansible-playbook test.yml -e "hosts=www user=magedu"
```

>`-e` 是`ansible-playbook`命令的选项，表示要传递变量。
 `"hosts=www user=magedu"` 是传递的变量参数。
 `hosts=www` 指定了`hosts`变量的值为`www`。
 `user=magedu` 指定了`user`变量的值为`magedu`

编写yaml文件
```yaml
- hosts: websrvs
  remote_user: root
 
  tasks:
    - name: install package
      yum: name={{ pkName }}
    - name: start service
      service : name={{ pkName }} state=started enabled=yes
```

在命令行执行时，使用-e参数，后面接要传递的变量
ansible-playbook -e 'pkName=vsftpd' app.yml

检查ftp服务是否安装成功：通过检查21端口是否开始监听
ansible websrvs -m shell -a 'ss -ntl | grep 21'

卸载刚才安装的服务
ansible websrvs -m shell -a 'yum -y remove vsftpd'

# 在独立的变量文件中定义

![[Pasted image 20230825160422.png]]

![[Pasted image 20230825161139.png]]

![[Pasted image 20230825161538.png]]


在自定义的yml文件中定义变量，在playbook中使用vars_files关键字引用

示例：安装httpd，创建log文件夹 创建变量文件，my_vars.yml（不是剧本，只是一个yaml文件）

```yaml
var1: httpd
var2: vsftpd
```

创建剧本文件app.yml ，调用上面的变量

```yaml
- hosts: app
  remote_user: root
  vars_files:
    - my_vars.yml

  tasks:
    - name: install package httpd
      yum: name={{ var1 }}
    - name: create log file vsftpd
      file: name=/opt/{{ var2 }}.log state=touch
```

执行

```
ansible-playbook -C app.yml
ansible-playbook app.yml
```

检查安装情况

```
ansible app -m shell -a 'rpm -q httpd' # 查询httpd的安装版本情况
ansible app -m shell -a 'ls -l /opt' # 查看目录创建情况
```

卸载&删除文件夹

```
ansible websrvs -m shell -a 'yum -y remove httpd'
ansible websrvs -m shell -a 'rm -rf /opt/vsftpd.log'
```
# 优先级

命令指定>playbook定义>主机清单主机变量>主机清单分组变量