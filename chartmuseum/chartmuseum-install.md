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
```
