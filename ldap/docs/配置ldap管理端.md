# 配置管理端

- 安装apache 和php环境

```bash
yum install php php-pear php-mbstring ntpdate httpd php-ldap -y     
#php-ldap是让php程序连接ldap的组件
systemctl start httpd
systemctl enable httpd
```

- 调整apache和php的配置文件，修改apache的默认首页类型，将index.php加入其中

```bash
vim /etc/httpd/conf/httpd.conf +164
<IfModule dir_module>   
DirectoryIndex  index.php
</IfModule>
```

- 重启apache

```bash
systemctl restart httpd
```

- 修改php.ini的时区

```bash
vim /etc/php.ini 
date.timezone = "Asia/Shanghai"
```

- 配置phpldapadmin

```bash
wget https://nchc.dl.sourceforge.net/project/phpldapadmin/phpldapadmin-php5/1.2.3/phpldapadmin-1.2.3.tgz
tar xf phpldapadmin-1.2.3.tgz 
mv phpldapadmin-1.2.3 /var/www/html/phpldapadmin
cp /var/www/html/phpldapadmin/config/config.php.example /var/www/html/phpldapadmin/config/config.php.example.bak
mv /var/www/html/phpldapadmin/config/config.php.example /var/www/html/phpldapadmin/config/config.php
```

- 修改配置文件，填写端口，地址和RootDN

```bash
vi /var/www/html/phpldapadmin/config/config.php
$servers->setValue('server','host','127.0.0.1');
$servers->setValue('server','port',389);
$servers->setValue('server','base',array('dc=fzport,dc=com'));
```

- 去浏览器验证登录

  [http://172.16.26.3/phpldapadmin]

  登录名为DN：cn=admin,dc=fzport,dc=com
  密码为第二次设置的密码：undead@666666