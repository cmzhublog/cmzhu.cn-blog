## supervisor 安装方案

#### pip 安装方案

1、适配不支持yum 安装的服务器,使用pip3 进行安装

```bash
$ pip3 install supervisor
```

2、创建对应的配置文件

```bash
$ echo_supervisord_conf | sudo tee /etc/supervisord.conf  
```

3、使用systemd 管理supervisor

```bash
 cat > /etc/systemd/system/supervisord.service <<EOF
[Unit]
Description=Supervisor process control system
After=network.target  
[Service]
Type=forking          
ExecStart=/usr/local/bin/supervisord -c /etc/supervisord.conf  
ExecStop=/usr/local/bin/supervisord/supervisorctl shutdown  
ExecReload=/usr/local/bin/supervisorctl reload  
User=root             
Restart=on-failure    
RestartSec=10s         
[Install]
WantedBy=multi-user.target
EOF
```

4、 重载systemd 配置,并配置开机自启

```bash
$ systemctl daemon-reload
$ systemctl enable --now supervisord
```

5、 优化supervisor 子配置管理

```bash
$ mkdir -p /etc/supervisord.d/
$ sed -i "s/;[include]/[include]/g" /etc/supervisord.conf 
$ sed -i "s/;files = /etc/supervisord.d/*.ini/files = /etc/supervisord.d/*.ini/g" /etc/supervisord.conf 


```

6、 重启服务

```bash
$ systemctl restart supervisord
```

