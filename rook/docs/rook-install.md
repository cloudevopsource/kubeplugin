# rook 安装指南



## 安装前规划



## 安装前的准备


+ 所有节点开启ip_forward
``` bash
cat <<EOF >  /etc/sysctl.d/ceph.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

+ 设置master运行pod(重要)

``` bash
#Master Node不参与工作负载，所以Taints: node-role.kubernetes.io/master:NoSchedule
#通过以下命令进行查看
kubectl describe node k8scloud1.frcloud.io | grep Taints 状态分别为node-role.kubernetes.io/master:NoSchedule 或<none>
#如果希望将k8s-master也当作Node节点使用，可以执行如下命令,其中k8s-master是主机节点hostname：
kubectl taint node k8scloud1.frcloud.io node-role.kubernetes.io/master-
kubectl taint node k8scloud2.frcloud.io node-role.kubernetes.io/master-
kubectl taint node k8scloud3.frcloud.io node-role.kubernetes.io/master-
#如果要恢复Master Only状态，执行如下命令：
kubectl taint node k8scloud1.frcloud.io node-role.kubernetes.io/master=:NoSchedule
```

## 安装rook

+ 下载最新rook软件包
``` bash
cd /stage
git clone https://github.com/rook/rook.git

```
+ 配置rook
``` bash
cd cluster/examples/kubernetes/ceph
kubectl create -f common.yaml
kubectl create -f operator.yaml
```

+ 添加帐户控制权限
``` bash
cd rook
cd cluster/examples/kubernetes/ceph
kubectl apply -f common.yaml
```

+ 部署Rook Operator
``` bash
cd rook
cd cluster/examples/kubernetes/ceph
kubectl apply -f operator.yaml
```

+ 编辑 cluster.yaml,给节点打标签
``` bash
#运行ceph-mon的节点打上：ceph-mon=enabled
kubectl label nodes {k8scloud1.frcloud.io,k8scloud2.frcloud.io,k8scloud3.frcloud.io} ceph-mon=enabled
#运行ceph-osd的节点，也就是存储节点，打上：ceph-osd=enabled
kubectl label nodes {k8scloud1.frcloud.io,k8scloud2.frcloud.io,k8scloud3.frcloud.io} ceph-osd=enabled
#运行ceph-mgr的节点，打上：ceph-mgr=enabled,mgr只能支持一个节点运行，这是ceph跑k8s里的局限
kubectl label nodes k8scloud1.frcloud.io ceph-mgr=enabled
```

+ 开始部署ceph
``` bash
kubectl apply -f cluster.yaml
# cluster会在rook-ceph这个namesapce创建资源,盯着这个namesapce的pod你就会发现，它在按照顺序创建Pod
kubectl -n rook-ceph get pod -o wide  -w
# 看到所有的pod都Running就行了,此时注意看一下pod分布的宿主机，跟我们打标签的主机是一致的
kubectl -n rook-ceph get pod -o wide
```


## 安装后的配置

+ 配置ceph dashboard
