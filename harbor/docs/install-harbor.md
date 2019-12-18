基本概念
CA(Certificate Authority)被称为证书授权中心，是数字证书发放和管理的机构。
根证书是CA认证中心给自己颁发的证书,是信任链的起始点。安装根证书意味着对这个CA认证中心的信任。
数字证书颁发过程一般为：用户首先产生自己的密钥对，并将公共密钥及部分个人身份信息传送给认证中心。认证中心在核实身份后，将执行一些必要的步骤，以确信请求确实由用户发送而来，然后，认证中心将发给用户一个数字证书，该证书内包含用户的个人信息和他的公钥信息，同时还附有认证中心的签名信息。
认证流程
https认证流程：

1、服务器生成一对密钥，私钥自己留着，公钥交给数字证书认证机构（CA）
2、CA进行审核，并用CA自己的私钥对服务器提供的公钥进行签名生成数字证书
3、在https建立连接时，客户端从服务器获取数字证书，用CA的公钥（根证书）对数字证书进行验证，比对一致，说明该数字证书确实是CA颁发的（得此结论有一个前提就是：客户端的CA公钥确实是CA的公钥，即该CA的公钥与CA对服务器提供的公钥进行签名的私钥确实是一对。），而CA又作为权威机构保证该公钥的确是服务器端提供的，从而可以确认该证书中的公钥确实是合法服务器端提供的。
注：为保证第3步中提到的前提条件，CA的公钥必须要安全地转交给客户端（CA根证书必须先安装在客户端），因此，CA的公钥一般来说由浏览器开发商内置在浏览器的内部。于是，该前提条件在各种信任机制上，基本保证成立。

制作自己的根证书及数字证书
通过CA生成数字证书比较很麻烦，而且要收费贵，在公司设备与公司的服务器通信时可以采用自己生成的数字证书，但是需要将数字证书导入到设备中。











sudo mkdir -p /usr/local/harbor/cert mkdir
sudo mkdir -p /usr/local/harbor/bin 
sudo cd /usr/local/harbor/bin
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
mv cfssl_linux-amd64 /usr/local/harbor/bin/cfssl

wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
mv cfssljson_linux-amd64 /usr/local/harbor/bin/cfssljson

wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
mv cfssl-certinfo_linux-amd64 /usr/local/harbor/bin/cfssl-certinfo

chmod +x /usr/local/harbor/bin/*
export PATH=/usr/local/harbor/bin:$PATH



创建根证书 (CA)
CA 证书是集群所有节点共享的，只需要创建一个 CA 证书，后续创建的所有证书都由它签名。

创建配置文件
CA 配置文件用于配置根证书的使用场景 (profile) 和具体参数 (usage，过期时间、服务端认证、客户端认证、加密等)，后续在签名其它证书时需要指定特定场景。

cd /usr/local/harbor/cert
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "harbor": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF


创建证书签名请求文件
cd /usr/local/harbor/cert
cat > ca-csr.json <<EOF
{
  "CN": "harbor",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "harbor",
      "OU": "4Paradigm"
    }
  ]
}
EOF



CN：Common Name，kube-apiserver 从证书中提取该字段作为请求的用户名 (User Name)，浏览器使用该字段验证网站是否合法；
O：Organization，kube-apiserver 从证书中提取该字段作为请求用户所属的组 (Group)；
kube-apiserver 将提取的 User、Group 作为 RBAC 授权的用户标识；





生成 CA 证书和私钥
cd /opt/k8s/work
cfssl gencert -initca ca-csr.json | cfssljson -bare ca

ls ca*





创建 harbor nginx 服务器使用的 x509 证书
创建 harbor 证书签名请求：
export NODE_IP=172.16.200.1# 当前部署 harbor 的节点 IP



cat > harbor-csr.json <<EOF
{
  "CN": "harbor",
  "hosts": [
    "127.0.0.1",
    "172.16.26.1",
    "harbor.frcloud.io"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "harbor",
      "OU": "4Paradigm"
    }
  ]
}
EOF




cfssl gencert -ca=/etc/harbor/cert/ca.pem \
  -ca-key=/etc/harbor/cert/ca-key.pem \
  -config=/etc/harbor/cert/ca-config.json \
-profile=harbor harbor-csr.json | cfssljson -bare harbor

$ ls harbor*
harbor.csr  harbor-csr.json  harbor-key.pem harbor.pem

$ sudo mkdir -p /etc/harbor/ssl
$ sudo mv harbor*.pem /etc/harbor/ssl
$ rm harbor.csr  harbor-csr.json

sudo EXTERNAL_URL="https://gitlab.frcloud.io" yum install -y gitlab-ce



2.3.4功能验证
https://172.16.200.1/harbor
默认账户：admin   Harbor12345

客户端：
$ sudo mkdir -p /etc/docker/certs.d/172.16.200.1
$ sudo cp /etc/harbor/cert/ca.pem /etc/docker/certs.d/172.16.200.1/ca.crt
$ docker login 192.168.40.136
Username: admin
Password:
