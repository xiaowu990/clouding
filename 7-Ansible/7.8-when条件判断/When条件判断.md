# 基本概念

![[Pasted image 20230825221618.png]]

![[Pasted image 20230825222644.png]]

# 详细介绍
> [!summary] 详细介绍

when语句中可以使用facts或playbook中定义的变量

语法：
* 在task后添加when子句即可使用条件测试；when语句支持Jinja2表达式语法
* 简单示例如下：当系统属于红帽系列，执行
```yaml
tasks:
  - name: "shutdown RedHat flavored systems"
    command: /sbin/shutdown -h now
    when: ansible_os_family == "RedHat"
```

when语句中还可以使用Jinja2的大多"filter"，如要忽略此前某语句的错误并基于其结果(failed或者success)运行后面指定的语句

```yaml
tasks:
  - command: /bin/false
    register: result
    ignore_errors: True
  - command: /bin/something
    when: result|failed
  - command: /bin/something_else
    when: result|success
  - command: /bin/still/something_else
    when: result|skipped
```

## 应用实例

* 不同的系统版本使用不同的操作

```yaml
- hosts: websrvs
  remote_user: root
  tasks:
    - name: add group nginx
      tags: user
      user: name=nginx state=present
    - name: add user nginx
      user: name=nginx state=present group=nginx
    - name: Install Nginx
      yum: name=nginx state=present
    - name: restart Nginx
      service: name=nginx state=restarted
      when: ansible_distribution_major_version == "6"
```

* 不同的系统版本使用不同的配置文件
```yaml
tasks:
  - name: install conf file to centos7
    template: src=nginx.conf.c7.j2 dest=/etc/nginx/nginx.conf
    when: ansible_distribution_major_version == "7"
  - name: install conf file to centos6
    template: src=nginx.conf.c6.j2 dest=/etc/nginx/nginx.conf
    when: ansible_distribution_major_version == "6"

```



