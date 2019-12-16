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
kubectl create -f cluster-test.yaml
```
