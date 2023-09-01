# 运维的发展以及背景
常用运维工具
saltstack，Puppet，Fabric，Chef....

运维自动化发展历程
![[Pasted image 20230810190333.png]]
类比

![[Pasted image 20230810190344.png]]




# 相关网址
ansible中文官网
http://www.ansible.com.cn/
ansible官方文档
[Ansible Documentation](https://docs.ansible.com/)

# 基本概念
定义：Ansible是一款自动化运维工具，基于Python开发，集合了众多运维工具（puppet、cfengine、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。==Ansible也是一种安装在控制节点上的*无代理*自动化工具。==

特点
* 基于SSH：Ansible使用SSH协议与被管理的主机进行通信，无需在被管理主机上安装任何代理软件或客户端。
* 基于剧本：Ansible使用YAML格式的剧本（Playbooks）来定义任务和配置。剧本是一种可读性强的文本文件，描述了一系列任务和目标状态。
* 声明性语言（YAML）：Ansible使用声明性语言来描述系统的期望状态，而不是编写一系列命令来实现特定的操作。这使得配置管理更加简单和可维护。
* 模块化架构： Ansible采用模块化的架构，提供了丰富的内置模块，用于执行各种操作，如文件管理、软件包管理、用户管理等。同时，还可以编写自定义模块来满足特定需求。
* 无代理：无代理意味着在被管理的主机上不需要安装任何额外的代理程序。传统上，一些自动化工具需要在被管理主机上安装特定的代理程序或客户端，以便能够与控制节点进行通信和执行操作。这些代理程序通常需要在每个被管理主机上进行安装和配置，以建立与控制节点的连接并接受指令。
* 广泛的支持：Ansible可以管理各种类型的系统，包括Linux、Windows、网络设备等。它还提供了与各种云平台和容器平台的集成，如AWS、Azure、Docker等。
# Ansible特性
## 为什么使用Ansible
- 提高工作的效率
- 提高工作准确度
- 减少维护的成本
- 减少重复性工作
## Ansible的优点

管理端不需要启动服务程序（no server）
管理端不需要编写配置文件（/etc/ansible/ansible.cfg）
受控端不需要安装软件程序（libselinux-python）
受控端不需要启动服务程序（no agent）
服务程序管理操作模块众多（module）
利用剧本编写来实现自动化（playbook）
支持sudo 、普通用户
## Ansible的功能
- 可以实现批量系统操作配置
- 可以实现批量软件服务部署
- 可以实现批量文件数据分发
- 可以实现批量系统信息收集
# Ansible架构
Ansible是基于模块工作的，本身没有批量部署的能力。真正具有批量部署的是Ansible所运行的模块，Ansible只是提供一种框架。主要包括：

1. 连接插件connection plugins：负责和被监控端实现通信
2. host inventory：指定操作的主机，是一个配置文件里面定义监控的主机
3. 各种模块核心模块、command模块、自定义模块
4. 借助于插件完成记录日志邮件等功能
5. playbook：剧本执行多个任务时，非必需可以让节点一次性运行多个任务

![[Pasted image 20230810190752.png]]
# Ansible的组成
一下是ansible的五个核心组成部分

* Ansible：ansible核心部分，可理解为是ansible命令工具，其为核心执行工具
* Modules：Ansible执行命令的功能模块，包括 Ansible 自带的核心模块及自定义模块
* Plugins：插件提供了对Ansible功能的扩展和定制，包括连接插件、邮件插件，该功能不常用。
* Playbooks：剧本是用于定义和描述Ansible任务集的配置文件。它以YAML格式编写，包含一系列任务、主机组、变量和配置选项等。通过编写剧本，可以实现对目标主机的自动化部署和配置管理，并按照指定的顺序执行任务。
* Inventory：定义 Ansible 管理主机的清单


# Ansible工作原理

![[Pasted image 20230810192206.png]]
# Ansible命令执行来源

- USER 普通用户，即SYSTEM ADMINISTRATOR
- PLAYBOOKS：任务剧本（任务集），编排定义Ansible任务集的配置文件，由Ansible顺序依次执行，通常是JSON格式的YML文件
- CMDB（配置管理数据库） API 调用
- PUBLIC/PRIVATE CLOUD API调用
- USER-> Ansible Playbook -> Ansibile

# Ansible实现管理任务的两种方式
* Ad-Hoc（即时任务）
使用==单条Ansible命令==来执行==临时任务==的方式。
Ad-Hoc命令的格式为
`ansible <hosts> -m <module> -a '<arguments>'`，
   * `<hosts>`表示目标主机或主机组，
   * `<module>`表示要执行的模块，
   * `<arguments>`表示模块的参数。
>这种方式适用于快速、简单的操作和临时需求



* Ansible Playbook（剧本）
Ansible Playbook是一种以编排任务集为基础的管理方式，==适用于长期规划和大型项目的场景。==通过编写Ansible Playbook，可以定义和描述一系列任务和配置，以实现自动化的部署和配置管理。

执行过程
   * Ansible Playbook是一种以编排任务集为基础的管理方式，适用于长期规划和大型项目的场景。
   * 通过编写Ansible Playbook，可以定义和描述一系列任务和配置，以实现自动化的部署和配置管理。

# 注意事项
ansible不是服务,不会一直启动,只是需要的时候启动
主控端

- 执行ansible的主机一般称为主控端，中控，master或堡垒机
- 主控端Python版本需要2.6或以上
- windows不能做为主控端

被控端

- 被控端Python版本小于2.4需要安装python-simplejson
- 被控端如开启SELinux需要安装libselinux-python