
# 准备工作

## 系统环境

**release：centos7.7_1908**

**ip：172.16.26.3**

**域名：ldap.fzport.com**

**hostname:openldap-server**

## 内核升级

+ 升级软件包
``` bash
yum update
```

+ 安装内核源仓库
``` bash
#Import the public key:
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum install https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm 
```

+ 查看指定的内核
``` bash
yum --enablerepo=elrepo-kernel  list  |grep kernel*
```
+ 移除旧内核，并安装新内核

``` bash
rpm -qa | grep kernel
yum remove -y  kernel-devel*
yum remove -y kernel-tools*
yum remove -y kernel-header*
yum --enablerepo=elrepo-kernel install  kernel-ml kernel-ml-devel kernel-ml-headers kernel-ml-tools kernel-ml-tools-libs kernel-ml-tools-libs-devel  -y
```
+ 设置默认启动项
``` bash
grub2-set-default 0
```

+ 更新grub.cfg
``` bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```
+ 重新切换到新内核版本
``` bash
reboot
#删除旧内核版本
yum remove -y kernel-3.10.0*
#再次查看当前内核版本
uname -r
rpm -qa | grep kernel
```



## 服务及参数设置

+ 关闭 防火墙
``` bash
systemctl stop firewalld
systemctl disable firewalld
```
+ 关闭 SeLinux
``` bash
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

## 将域名解析写入到hosts

```bash
echo "172.16.26.3    ldap.fzport.com" >> /etc/hosts
```

# 安装

- yum安装openldap，并采用`cn=config`方式（修改配置会立即生效，不用重启slapd）

```bash
yum install -y openldap openldap-clients openldap-devel openldap-servers compat-openldap migrationtools
```

- 准备BDB数据库文件

```bash
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
chown ldap:ldap /var/lib/ldap/DB_CONFIG
```

- 启动并加入开机自启动

```bash
systemctl start slapd
systemctl enable slapd
```

# 基础配置

- 生成openldap的管理密码undead@666（记下来，下面将用到）


```bash
[root@openldap-server ~]# slappasswd

New password: 

Re-enter new password: 

{SSHA}5KqLmqUXoiq/I3nDByp2NKDNjc4STyjW
```

- 编写ldif文件（填入上面生成的ssha为olcRootPW密码）


```bash
vim chrootpw.ldif 

dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}5KqLmqUXoiq/I3nDByp2NKDNjc4STyjW #填入上面生成的ssha
```

- 导入ldif文件

```bash
ldapadd -Y EXTERNAL -H ldapi:/// -f chrootpw.ldif
```

- 导入基础的Schemas （openldap的基础模块在/etc/openldap/schema/目录里面）

```bash
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif 
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
```

# 配置openldap的条目

- 先准备一个openldap 根DN的管理密码(undead@666666)，然后设置你的RootDN名字在openldap的数据库中

```bash
[root@openldap-server ~]# slappasswd
New password: Re-enter 
new password: {SSHA}sA4tp2fDiU/DVMfYTc65ugQDqaNyt3ai
```

- 编写RootDN的ldif文件（cn=admin,dc=fzport,dc=com）

```yaml
# vim chdomain.ldif

# replace to your own domain name for "dc=***,dc=***" section

# specify the password generated above for "olcRootPW" section

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=admin,dc=fzport,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=fzport,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=admin,dc=fzport,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}6lzO62ak2NCy9iF5pYHHL9j3blQvo7/m

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=admin,dc=fzport,dc=com" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=admin,dc=fzport,dc=com" write by * read
```

- 导入定义RootDN的ldif文件

```bash
ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif
```

- 编写基础的domain条目的ldif

```yaml
#vim  basedomain.ldif

# replace to your own domain name for "dc=***,dc=***" section

dn: dc=fzport,dc=com
objectClass: top
objectClass: dcObject
objectclass: organization
o: fzport com
dc: fzport

dn: cn=Manager,dc=fzport,dc=com
objectClass: organizationalRole
cn: Manager
description: Directory Manager

dn: ou=People,dc=fzport,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=fzport,dc=com
objectClass: organizationalUnit
ou: Group
```

- 导入基础的domain条目文件

```bash
ldapadd -x -D cn=admin,dc=fzport,dc=com -W -f basedomain.ldif    #这里会要求输入openldap数据库的密码，也就是设置的第二个密码
```

# 验证是否正常启动

```bash
#验证查看slapd服务是否启动，并监听389端口
ps -ef |grep slapd
ss -tnl |grep 389
```

\#查看服务器openldap目录树信息

```bash
ldapsearch -x -b "dc=fzport,dc=com" -H ldap://127.0.0.1
```

