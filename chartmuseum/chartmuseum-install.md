# chartmuseum 安装指南





+ 下载安装包
``` bash
curl -LO https://s3.amazonaws.com/chartmuseum/release/latest/bin/linux/amd64/chartmuseum
chmod +x chartmuseum
cp chartmuseum /usr/local/bin
```

+ systemd方式启动chartmuseum

``` bash
# cat /etc/systemd/system/chartmuseum.service
[Unit]
Description=chartmuseum
Requires=network-online.target
After=network-online.target

[Service]
EnvironmentFile=/etc/chartmuseum/chartmuseum.config
User=root
Restart=allways
ExecStart=/usr/local/bin/chartmuseum $ARGS
ExecStop=/usr/local/bin/chartmuseum step-down

[Install]
WantedBy=multi-user.target
```

+ EnvironmentFile的/etc/chartmuseum/chartmuseum.config配置

``` bash
ARGS=\
--port=8889 \
--storage="local" \
--storage-local-rootdir="/var/lib/chartmuseum/chartstorage" \
--log-json \
--basic-auth-user=admin \
--basic-auth-pass="Undead@666"

#--port： chartmuseum服务监听端口
#--storage： local表示使用本地存储
#--storage-local-rootdir: 本地存储点路径，helm push chart的存储路径
#--log-json： 日志显示为json格式
#--basic-auth-user： 用户名（使用基本的认证方式，用户名+密码，使用证书方式参照点我)
#--asic-auth-pass: 密码 （chartmuseum服务起来后，后续给helm添加repo时需要加上--username xxx --password ***）
```
+ 启动服务
``` bash
systemctl start chartmuseum
systemctl status chaetmuseum
```
+ 添加chartmuseum 到helm repo
``` bash
helm repo add chartmuseum http://192.168.4.32:9090 --username  admin --password  admin
```
