# 基本介绍
😉
- 在task中使用with_items给定要迭代的元素列表
    
- 对迭代项的引用，固定变量名为 `item`，只能写item
    
- 列表格式：字符串，字典

# 案例1-迭代创建文件、安装软件
🤑
> [!seealso] 
> 示例：创建多个文件、安装多个包


![[Pasted image 20230825225438.png]]

以上就是循环创建三个文件，循环安装三个软件包

执行命令

```
ansible-playbook -C app.yml
ansible-playbook app.yml
```

验证

```
ansible app -m shell -a 'rpm -q htop sl hping3'
```

# 案例2-创建多个组
```yaml
- hosts: app
  remote_user: root
  
  tasks:
    - name: create group
      group: name={{ item }}
      when: ansible_distribution_major_version == '7'
      with_items:
        - g1
        - g2
        - g3
```

#  案例3-创建多个用户
```yaml
- hosts: app
  remote_user: root
  
  tasks:
    - name: add users
      user: name={{ item }} state=present groups=wheel 
      when: ansible_distribution_major_version == '7'
      with_items:
        - user1
        - user2
```

# 案例4-迭代嵌套子变量
创建不同用户到不同的组

```yaml
- hosts: app
  remote_user: root
  
  tasks:
    - name: add users
      user: name={{ item.name }} state=present groups={{ item.group }} 
      when: ansible_distribution_major_version == '7'
      with_items:
        - { name: 'user1', group: 'wheel' }
        - { name: 'user2', group: 'root' }
```

