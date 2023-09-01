![[Pasted image 20230818200004.png]]

浏览器输入以下网址即可下载
https://github.com/prometheus/pushgateway/releases/download/v1.2.0/pushgateway-1.2.0.linux-amd64.tar.gz

tar xf pushgateway-1.2.0.linux-amd64.tar.gz
mv pushgateway-1.2.0.linux-amd64 /usr/local/pushgateway

增加systemd启动方式
vim /usr/lib/systemd/system/pushgateway.service
输入以下内容
```shell
[Unit]
Description=prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
WorkingDirectory=/usr/local/pushgateway
ExecStart=/usr/local/pushgateway/pushgateway \
           --web.enable-admin-api \
           --persistence.file="pushfile.txt" \
           --persistence.interval=10m
[Install]
WantedBy=multi-user.target
```

启动：systemctl start pushgateway
查看端口: ps -ef | grep pushgateway
![[Pasted image 20230818202217.png]]


