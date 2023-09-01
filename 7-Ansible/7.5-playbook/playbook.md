# 基本概念
## 相关概念
定义：playbook采用YAML语言编写的文本文件，是由一个或多个按特定顺序运行的play组成的列表。

**play（剧本）**
- Play是一个高级别的概念，它代表一组相关的任务的有序集合，用于实现特定的目标或配置。
- ==一个Play由一个或多个Task组成===，可以定义在一个Playbook文件中。
- Play可以指定在哪些主机或主机组上执行任务，可以定义变量、条件和循环，以及其他Play级别的配置选项。
- Play具有明确的执行顺序，并且可以根据需要进行控制和组织。

**task（任务）**
- Task是Play中的最小执行单元，==它代表一个具体的操作或命令，还是调用模块==，在远程主机上执行。
- Task通常使用模块来实现特定的操作，如文件管理、软件包安装、服务管理等。
- Task可以包含多个操作，每个操作使用模块的特定参数来定义。
- Task可以定义在Play的`tasks`部分中，按照顺序执行

两者区别
**Play是对一系列任务的组织和控制，而每个Task代表一个具体的操作，如安装软件包、配置文件、启动服务等**
- Play是一个更高级别的概念，代表了一组任务的集合，而Task是具体的执行单元。
- 一个Play可以包含多个Task，按照指定的顺序依次执行。
- Play可以定义在Playbook文件中，而Task定义在Play的`tasks`部分中。
- Task通常使用模块来执行具体的操作，而Play可以定义变量、条件、循环等高级功能。


案例演示
```yaml
- name: Multi-Playbook Example
  hosts: all
  
  tasks:
    - name: Task in Play 1
      debug:
        msg: "This is Task 1 in Play 1"
  
- name: Play 2
  hosts: web_servers
  
  tasks:
    - name: Task in Play 2
      debug:
        msg: "This is Task 1 in Play 2"
  
    - name: Task 2 in Play 2
      debug:
        msg: "This is Task 2 in Play 2"
  
- name: Play 3
  hosts: database_servers
  
  tasks:
    - name: Task in Play 3
      debug:
        msg: "This is Task 1 in Play 3"
        ```



![[Pasted image 20230813182846.png]]

## 流程
- 用户通过ansible命令直接调用yml语言写好的playbook
- playbook由多条play组成
- 每条play都有相关task相对应的操作，调用模块modules
- 应用在主机清单上,通过ssh远程连接，从而控制远程主机或者网络设备

![[Pasted image 20230813202050.png]]

## 加密
由于playbook中可能会包含敏感信息，需要进行加密操作，使用ansible-vault命令进行加密操作
对一个hello.yml进行加密。

# playbook核心元素

Hosts ：执行的远程主机列表
Tasks ：任务集
Varniables ：内置变量或自定义变量在playbook中调用
Templates ：模板，即使用模板语法的文件，比如配置文件等
Handlers ：和notify结合使用，由特定条件触发的操作，满足条件方才执行，否则不执行
tags ：标签，指定某条任务执行，用于选择运行playbook中的部分代码。

### hosts
- hosts用于指定要执行指定任务的主机，须事先定义在主机清单中
- 例：hosts: websrvs:dbsrvs

remote_user:`remote_user` 
> 表示本机以root身份连接远程机器


用于host和task中， 也可以通过指定其通过sudo的方式在远程主机上执行任务，其可用于play全局或某任务，此外，甚至可以在sudo时使用sudo_user指定sudo时切换的用户

示例
```yaml
- hosts: websrvs
  remote_user: root   # (可省略,默认为root)  表示本机以root身份连接远程机器
  
  tasks:    # 指定任务
    - name: test connection # task（任务）的描述
      ping:
        remote_user: s
        sudo: yes           # 默认sudo为root
        sudo_user:wang      # sudo为wang
```

### task
- playbook的主体部分是task list
    
    - task list中的各任务按次序逐个在hosts中指定的所有主机上执行
    - ==在所有主机上完成第一个任务后，再开始第二个任务==
   
- task的目的：使用指定的参数执行模块
    
    - 在模块参数中可以使用变量
    - 模块执行是幂等的，多次执行是安全的，其结果均一致
    
- 每个task都有name（==就是说明任务时干啥的==），用于playbook的执行结果输出
    
    - 建议其内容能清晰地描述任务执行步骤
    - 如果未提供name，则action的结果将用于输出
    
- 两种书写格式
    
    - action: module arguments
    - module: arguments   *推荐使用*
- 注意：==shell和command模块后面跟命令==，而非key=value
    
- 某任务的状态在运行后为changed时，可通过"notify"通知给相应的handlers
    
- 任务可以通过"tags"打标签，可在ansible-playbook命令上使用-t指定进行调用


### 示例

创建一个文件，以及用户，拷贝html文件，启动一个服务

```yaml
- hosts: wbsr
  remote_user: root
  
  tasks:
    - name: create new file # 描述
      file: name=/data/newfile state=touch # 模块名: 模块对应的参数
    - name: create new user
      user: name=test2 system=yes shell=/sbin/nologin
    - name: install package
      yum: name=httpd
    - name: copy html
      copy: src=/var/www/html/index.html dest=/var/www/html/
    - name: start service
      service: name=httpd state=started enabled=yes
```

实操如下
1. 先使用yum install httpd安装httpd，
2. rpm -ql httpd。找到最底层的/var/www/html
3. echo welcome to magetu > /var/www/html/index.html
4. 先检查ansible-playbook -C p.yml
5. 开始执行ansible-playbook p.yml
![[Pasted image 20230824170008.png]]

![[Pasted image 20230824170356.png]]



某些情况下忽略错误
如果命令或脚本的退出码不为零，可以使用如下方式替代；
==针对有些命令执行出错了，但是不影响，可以跳过的情况使用==


```yaml
tasks:
  - name: run this command and ignore the result
    shell: /usr/bin/somecommand || /bin/true
    
# 或者使用ignore_errors来忽略错误信息
tasks:
  - name: run this command and ignore the result
    shell: /usr/bin/somecommand
    ignore_errors: True  #忽略错误
```

### 运行playbook
格式：ansible-playbook <filename.yml> ... [options]

常用参数
--check -C            只检测可能会发生的改变，但不真正执行操作 
                     (*只检查语法*,如果执行过程中出现问题,-C无法检测出来)
                      (执行playbook生成的文件不存在,后面的程序如果依赖这些文件,也会导致检测失败)
--list-hosts           列出运行任务的主机
--list-tags            列出tag  (列出标签)
--list-tasks           列出task (列出任务)
--limit 		            主机列表 只针对主机列表中的主机执行
-v -vv -vvv            显示过程

>ansible-playbook hello.yml -C 只检测
 ansible-playbook hello.yml --list-hosts  显示运行任务的主机
 ansible-playbook hello.yml --limit websrvs/ip  限制主机（表示只在特定的主机上运行）

相关示例
安装httpd服务
```yaml
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: Install httpd
      yum: name=httpd state=present
    - name: Install configure file
      copy: src=files/httpd.conf dest=/etc/httpd/conf/
    - name: start service
      service: name=httpd state=started enabled=yes
```

### handlers与notify

==`notify`关键字是可选的，它提供了一种将任务与处理程序关联起来的方式，以实现任务完成后执行特定操作的需求。如果任务没有指定`notify`关键字，相关的处理程序将不会被触发。==

![[Pasted image 20230814091037.png]]

**通知Handlers：** 任务可以使用`notify`关键字指定要通知的处理程序。当任务执行成功后，Ansible会按照指定的顺序触发相关的处理程序。
```yaml
tasks:
  - name: Start the web server
    service:
      name: nginx
      state: started
    notify:
      - Reload configuration
      - Restart service
在上述示例中，当"Start the web server"任务成功完成后，它将通知"Reload configuration"和"Restart service"这两个处理程序。
```

**定义Handlers：** Handlers在Playbook中以与任务相似的方式定义。它们被放置在顶层的`handlers`部分，并使用`name`关键字命名。每个Handler都有一个唯一的名称，用于后续的通知
```yaml
handlers:
  - name: Reload configuration
    command: /path/to/reload_script.sh
  - name: Restart service
    service:
      name: nginx
      state: restarted
```

**Handlers的执行顺序：** Handlers按照它们在Playbook中定义的顺序进行执行。当多个处理程序被通知时，Ansible会按照它们在处理程序列表中的顺序依次触发它们。
```yaml
handlers:
  - name: Reload configuration
    command: /path/to/reload_script.sh
  - name: Restart service
    service:
      name: nginx
      state: restarted

在上述示例中，"Reload configuration"处理程序将在"Restart service"处理程序之前执行。
```

**Handlers的触发次数：** Handlers只有在被通知时才会执行。如果多个任务通知了相同的处理程序，那么该处理程序将只执行一次，并且在所有任务完成后触发。

```yaml
tasks:
  - name: Task 1
    service:
      name: nginx
      state: started
    notify:
      - Reload configuration
  - name: Task 2
    service:
      name: apache
      state: started
    notify:
      - Reload configuration
在上述示例中，无论"Task 1"和"Task 2"是否都通知了"Reload configuration"处理程序，该处理程序只会在所有任务完成后执行一次。
```

### tags
在Ansible Playbook中，"tags"是一种用于标记任务的机制。通过为任务定义标签，可以选择性地执行或跳过特定标签的任务，从而实现更细粒度的控制

格式：ansible-playbook ==-t== tagsname useradd.yml
> -t就表示后面接标签

定义标签
* 可以在任务级别使用"tags"关键字为任务定义一个或多个标签。标签可以是任意字符串，用空格或逗号分隔多个标签。
* 多个动作可以使用同一个标签
* 一个任务也可以定义多个标签

```yaml
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: Install httpd
      yum: name=httpd state=present
      tage: install 
    - name: Install configure file
      copy: src=files/httpd.conf dest=/etc/httpd/conf/
      tags: conf
    - name: start httpd service
      tags: service
      service: name=httpd state=started enabled=yes
```

执行
*指定执行install,conf 两个标签内的action* 
ansible-playbook –t install,conf httpd.yml
或者跳过指定标签的task
ansible-playbook play.yml --skip-tags install 

# Yaml
![[Pasted image 20230824212128.png]]

![[Pasted image 20230824212151.png]]

![[Pasted image 20230824212202.png]]

![[Pasted image 20230824212213.png]]

![[Pasted image 20230824212233.png]]

![[Pasted image 20230824212252.png]]

![[Pasted image 20230824212310.png]]


