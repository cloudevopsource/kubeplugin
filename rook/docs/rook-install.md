# rook 安装指南



## 安装前规划



## 下载最新rook软件包
``` bash
cd /stage
git clone https://github.com/rook/rook.git

```

``` bash
cd cluster/examples/kubernetes/ceph
kubectl create -f common.yaml
kubectl create -f operator.yaml


#编辑 cluster.yaml
给节点打标签
运行ceph-mon的节点打上：ceph-mon=enabled
kubectl label nodes {k8scloud1.frcloud.io,k8scloud2.frcloud.io,k8scloud3.frcloud.io} ceph-mon=enabled
运行ceph-osd的节点，也就是存储节点，打上：ceph-osd=enabled
kubectl label nodes {k8scloud1.frcloud.io,k8scloud2.frcloud.io,k8scloud3.frcloud.io} ceph-osd=enabled
运行ceph-mgr的节点，打上：ceph-mgr=enabled
#mgr只能支持一个节点运行，这是ceph跑k8s里的局限
kubectl label nodes k8scloud1.frcloud.io ceph-mgr=enabled



kubectl create -f cluster.yaml
```
