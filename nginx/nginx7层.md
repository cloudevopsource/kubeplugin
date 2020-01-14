# Nginx安装配置

## 1、安装

下载nginx

```bash
wget http://nginx.org/download/nginx-1.16.1.tar.gz
```

解压nginx

```bash
tar zxvf  nginx-1.16.1.tar.gz
```

为方便操作，此处将nginx目录重命名：

```bash
mv nginx-1.16.1 nginx; cd nginx/
```

进入nginx目录

```bash
./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_stub_status_module --with-http_gzip_static_module --with-pcre --with-stream --with-stream_ssl_module --with-stream_realip_module
```

## 编译安装

```bash
 make -j 4 && make install
```

## 配置nginx泛域名解析

vi /etc/nginx.conf

```bash
worker_processes 1;

events {
    worker_connections  1024;
}


http {


upstream hwcloudk8s {
        ip_hash;
        server 192.168.224.1:30080;
        server 192.168.224.2:30080;
        server 192.168.224.3:30080;
}
orker_processes 1;

events {
    worker_connections  1024;
}


http {


upstream hwcloudk8s {
        ip_hash;
        server 192.168.224.1:30080;
        server 192.168.224.2:30080;
        server 192.168.224.3:30080;
}


upstream localk8s {
        ip_hash;
        server 172.16.22.1:30080;
        server 172.16.22.2:30080;
        server 172.16.22.3:30080;
}



server {
    listen   80;
    server_name  *.dev.frcloud.io;
    location / {
        proxy_pass              http://hwcloudk8s;
        proxy_redirect          off;
        proxy_set_header        Host $host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        #root   /usr/share/nginx/html;
        #index  index.html index.htm;
    }
}

server {
    listen   80;
    server_name  *.frcloud.io;
    location / {
        proxy_pass              http://localk8s;
        proxy_redirect          off;
        proxy_set_header        Host $host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        #root   /usr/share/nginx/html;
        #index  index.html index.htm;
    }
}


}
```

## 配置nginx service

```bash
[Unit]
Description=nginx proxy
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=forking
ExecStartPre=/usr/local/nginx/sbin/nginx -c /etc/nginx.conf -t
ExecStart=/usr/local/nginx/sbin/nginx -c /etc/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -c /etc/nginx.conf -s reload
PrivateTmp=true
Restart=always
RestartSec=5
StartLimitInterval=0
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

## 开机自启nginx

```bash
systemctl enable nginx
```

## 启动nginx服务

```bash
systemctl start nginx
```

## 查看nginx状态

```bash
systemctl status nginx
```

```bash
● nginx.service - nginx proxy
   Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2020-01-14 01:17:00 CST; 15h ago
  Process: 5289 ExecStart=/usr/local/nginx/sbin/nginx -c /etc/nginx.conf (code=exited, status=0/SUCCESS)
  Process: 5288 ExecStartPre=/usr/local/nginx/sbin/nginx -c /etc/nginx.conf -t (code=exited, status=0/SUCCESS)
 Main PID: 5291 (nginx)
   CGroup: /system.slice/nginx.service
           ├─5291 nginx: master process /usr/local/nginx/sbin/nginx -c /etc/nginx.conf
           └─5293 nginx: worker process

Jan 14 01:17:00 nginx1.fzport.com systemd[1]: Stopped nginx proxy.
Jan 14 01:17:00 nginx1.fzport.com systemd[1]: Starting nginx proxy...
Jan 14 01:17:00 nginx1.fzport.com nginx[5288]: nginx: the configuration file /etc/nginx.conf syntax is ok
Jan 14 01:17:00 nginx1.fzport.com nginx[5288]: nginx: configuration file /etc/nginx.conf test is successful
Jan 14 01:17:00 nginx1.fzport.com systemd[1]: Started nginx proxy.
```

