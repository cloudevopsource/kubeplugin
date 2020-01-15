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

[root@jenkinsx harbor.frcloud.io]# helm repo list
NAME            URL                                                   
stable          https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
local           http://127.0.0.1:8879/charts                          
localchart      http://172.16.26.1:8879                               
chart           http://172.16.26.1:8879                               
chartmuseum     http://172.16.26.1:8889 
```


chartmuseum和curl的使用
上传
curl -u admin:admin  --data-binary "@demo-0.3.0.tgz" http://192.168.4.32:9090/api/charts
下载
curl -O   -u admin:admin http://192.168.4.32:9090/charts/demo-0.1.0.tgz
chartmuseum其他API
GET /index.yaml 得到chartmuseum的全部charts
[root@t32 demo]# curl http://192.168.4.32:9090/index.yaml -u admin:admin
apiVersion: v1
entries:
  demo:
  - apiVersion: v1
    appVersion: "1.0"
    created: "2019-09-25T21:05:34.55346099+08:00"
    description: A Helm chart for Kubernetes
    digest: 98220d606e571949c29175e51f384d75f38e306d5ad7ccf0f882a61c4183a983
    name: demo
    urls:
    - charts/demo-0.3.0.tgz
    version: 0.3.0
  - apiVersion: v1
    appVersion: "1.0"
    created: "2019-09-25T19:00:27.301961076+08:00"
    description: A Helm chart for Kubernetes
    digest: fa496288ee05d446699f26e82bba4d6eefd6bcb87e47b78fdd6ce3e682319142
    name: demo
    urls:
    - charts/demo-0.2.0.tgz
    version: 0.2.0
  - apiVersion: v1
    appVersion: "1.0"
    created: "2019-09-25T18:54:32.406748864+08:00"
    description: A Helm chart for Kubernetes
    digest: df107a069a1a5459800cc0ed2017c59efd6bf01701b1a96c89e3d9f3ed229e64
    name: demo
    urls:
    - charts/demo-0.1.0.tgz
    version: 0.1.0
generated: "2019-09-25T21:06:00+08:00"
serverInfo: {}
GET /charts/demo-0.2.0.tgz 下载charts中的demo
[root@t32 demo]# curl -O http://192.168.4.32:9090/charts/demo-0.2.0.tgz -u admin:admin
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2564    0  2564    0     0   605k      0 --:--:-- --:--:-- --:--:--  834k
POST /api/charts 上传一个新的chart版本
[root@t32 demo]# curl -X POST  --data-binary '@demo-0.3.0.tgz'  http://192.168.4.32:9090/api/charts -u admin:admin
{"saved":true}
DELETE /api/charts/<name>/<version> 删除一个chart版本
[root@t32 demo]# curl  -s -X DELETE  http://192.168.4.32:9090/api/charts/demo/0.3.0  -u admin:admin | jq
{
  "deleted": true
}
GET /api/charts 列出所有的charts
[root@t32 demo]# curl -s  http://192.168.4.32:9090/api/charts -u admin:admin | jq
{
  "demo": [
    {
      "name": "demo",
      "version": "0.3.0",
      "description": "A Helm chart for Kubernetes",
      "apiVersion": "v1",
      "appVersion": "1.0",
      "urls": [
        "charts/demo-0.3.0.tgz"
      ],
      "created": "2019-09-25T21:05:34.55346099+08:00",
      "digest": "98220d606e571949c29175e51f384d75f38e306d5ad7ccf0f882a61c4183a983"
    },
    {
      "name": "demo",
      "version": "0.2.0",
      "description": "A Helm chart for Kubernetes",
      "apiVersion": "v1",
      "appVersion": "1.0",
      "urls": [
        "charts/demo-0.2.0.tgz"
      ],
      "created": "2019-09-25T19:00:27.301961076+08:00",
      "digest": "fa496288ee05d446699f26e82bba4d6eefd6bcb87e47b78fdd6ce3e682319142"
    },
    {
      "name": "demo",
      "version": "0.1.0",
      "description": "A Helm chart for Kubernetes",
      "apiVersion": "v1",
      "appVersion": "1.0",
      "urls": [
        "charts/demo-0.1.0.tgz"
      ],
      "created": "2019-09-25T18:54:32.406748864+08:00",
      "digest": "df107a069a1a5459800cc0ed2017c59efd6bf01701b1a96c89e3d9f3ed229e64"
    }
  ]
}
GET /api/chatts/<name> 列出chart的所有版本
[root@t32 demo]# curl -s  http://192.168.4.32:9090/api/charts/demo -u admin:admin | jq
[
  {
    "name": "demo",
    "version": "0.3.0",
    "description": "A Helm chart for Kubernetes",
    "apiVersion": "v1",
    "appVersion": "1.0",
    "urls": [
      "charts/demo-0.3.0.tgz"
    ],
    "created": "2019-09-25T21:05:34.55346099+08:00",
    "digest": "98220d606e571949c29175e51f384d75f38e306d5ad7ccf0f882a61c4183a983"
  },
  {
    "name": "demo",
    "version": "0.2.0",
    "description": "A Helm chart for Kubernetes",
    "apiVersion": "v1",
    "appVersion": "1.0",
    "urls": [
      "charts/demo-0.2.0.tgz"
    ],
    "created": "2019-09-25T19:00:27.301961076+08:00",
    "digest": "fa496288ee05d446699f26e82bba4d6eefd6bcb87e47b78fdd6ce3e682319142"
  },
  {
    "name": "demo",
    "version": "0.1.0",
    "description": "A Helm chart for Kubernetes",
    "apiVersion": "v1",
    "appVersion": "1.0",
    "urls": [
      "charts/demo-0.1.0.tgz"
    ],
    "created": "2019-09-25T18:54:32.406748864+08:00",
    "digest": "df107a069a1a5459800cc0ed2017c59efd6bf01701b1a96c89e3d9f3ed229e64"
  }
]
GET /api/charts/<name>/<version> 对一个chart版本的描述
[root@t32 demo]# curl -s  http://192.168.4.32:9090/api/charts/demo/0.3.0 -u admin:admin | jq
{
  "name": "demo",
  "version": "0.3.0",
  "description": "A Helm chart for Kubernetes",
  "apiVersion": "v1",
  "appVersion": "1.0",
  "urls": [
    "charts/demo-0.3.0.tgz"
  ],
  "created": "2019-09-25T21:05:34.55346099+08:00",
  "digest": "98220d606e571949c29175e51f384d75f38e306d5ad7ccf0f882a61c4183a983"
}
GET / HTML welcome page
[root@t32 demo]# curl  http://192.168.4.32:9090/health
{"healthy":true}[root@t32 demo]# curl  http://192.168.4.32:9090/
{"error":"unauthorized"}[root@t32 demo]# curl  http://192.168.4.32:9090/ -u admin:admin
<!DOCTYPE html>
<html>
<head>
<title>Welcome to ChartMuseum!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to ChartMuseum!</h1>
<p>If you see this page, the ChartMuseum web server is successfully installed and
working.</p>

<p>For online documentation and support please refer to the
<a href="https://github.com/helm/chartmuseum">GitHub project</a>.<br/>

<p><em>Thank you for using ChartMuseum.</em></p>
</body>
</html>
GET /health return 200 OK
[root@t32 demo]# curl  http://192.168.4.32:9090/health
{"healthy":true}
