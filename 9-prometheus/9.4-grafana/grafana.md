# 安装Grafana
- 在 Prometheus 中，我们可以使用 Web 界面进行数据的查询和展示，但是展示效果不是很好；
- 所以我们这里使用 Grafana 来配合 Prometheus 使用。
下载：wget https://dl.grafana.com/oss/release/grafana-6.1.4-1.x86_64.rpm
本地安装：yum -y localinstall grafana-6.1.4-1.x86_64.rpm
启动：systemctl start grafana-server
查看进程是否存在：ps -ef | grep grafana
设置开机启动：systemctl enable grafana-server
查看是否在3000端口：netstat -lntup | grep 3000
默认的用户名和密码都是admin和admin，登录后会要求修改密码
 > 注意去开放防火墙3000端口
 
![[Pasted image 20230817181559.png]]

![[Pasted image 20230817181659.png]]

# Grafana接入prometheus数据源

添加数据源

![[Pasted image 20230817182146.png]]

![[Pasted image 20230817182225.png]]

![[Pasted image 20230817182334.png]]

import-导入模板
![[Pasted image 20230817184143.png]]

![[Pasted image 20230817184149.png]]

![[Pasted image 20230817184154.png]]

