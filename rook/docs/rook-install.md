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

[root@k8scloud1 stage]# kubectl get pods -n rook-ceph -o wide
NAME                                                             READY   STATUS      RESTARTS   AGE    IP               NODE                   NOMINATED NODE   READINESS GATES
csi-cephfsplugin-88clg                                           3/3     Running     0          3h9m   192.168.224.4    k8scloud4.frcloud.io   <none>           <none>
csi-cephfsplugin-hxzdl                                           3/3     Running     0          3h9m   192.168.224.5    k8scloud5.frcloud.io   <none>           <none>
csi-cephfsplugin-jhdql                                           3/3     Running     0          3h9m   192.168.224.3    k8scloud3.frcloud.io   <none>           <none>
csi-cephfsplugin-kzl5m                                           3/3     Running     0          3h9m   192.168.224.1    k8scloud1.frcloud.io   <none>           <none>
csi-cephfsplugin-nmr6w                                           3/3     Running     0          3h9m   192.168.224.2    k8scloud2.frcloud.io   <none>           <none>
csi-cephfsplugin-provisioner-56c8b7ddf4-mrd5f                    4/4     Running     0          3h9m   10.100.177.24    k8scloud3.frcloud.io   <none>           <none>
csi-cephfsplugin-provisioner-56c8b7ddf4-w7xd4                    4/4     Running     0          3h9m   10.100.72.16     k8scloud4.frcloud.io   <none>           <none>
csi-rbdplugin-4jz5h                                              3/3     Running     0          3h9m   192.168.224.4    k8scloud4.frcloud.io   <none>           <none>
csi-rbdplugin-8qd7d                                              3/3     Running     0          3h9m   192.168.224.1    k8scloud1.frcloud.io   <none>           <none>
csi-rbdplugin-hjxcz                                              3/3     Running     0          3h9m   192.168.224.5    k8scloud5.frcloud.io   <none>           <none>
csi-rbdplugin-lnbhw                                              3/3     Running     0          3h9m   192.168.224.2    k8scloud2.frcloud.io   <none>           <none>
csi-rbdplugin-provisioner-6ff4dd4b94-6l55d                       5/5     Running     0          3h9m   10.100.144.25    k8scloud2.frcloud.io   <none>           <none>
csi-rbdplugin-provisioner-6ff4dd4b94-xt62n                       5/5     Running     2          3h9m   10.100.238.148   k8scloud5.frcloud.io   <none>           <none>
csi-rbdplugin-vxwt4                                              3/3     Running     0          3h9m   192.168.224.3    k8scloud3.frcloud.io   <none>           <none>
rook-ceph-crashcollector-k8scloud1.frcloud.io-6b89cf7ff9-g7vfm   1/1     Running     0          3h8m   10.100.94.42     k8scloud1.frcloud.io   <none>           <none>
rook-ceph-crashcollector-k8scloud2.frcloud.io-85dd47cfd-p822v    1/1     Running     0          3h8m   10.100.144.28    k8scloud2.frcloud.io   <none>           <none>
rook-ceph-crashcollector-k8scloud3.frcloud.io-bff987d8f-x6wlb    1/1     Running     0          3h8m   10.100.177.29    k8scloud3.frcloud.io   <none>           <none>
rook-ceph-mds-myfs-a-d598db65d-cc4j9                             1/1     Running     0          40m    10.100.144.36    k8scloud2.frcloud.io   <none>           <none>
rook-ceph-mds-myfs-b-99dd4ff85-8j57x                             1/1     Running     0          40m    10.100.177.38    k8scloud3.frcloud.io   <none>           <none>
rook-ceph-mgr-a-59df5cb66c-rbjnl                                 1/1     Running     0          3h8m   10.100.94.40     k8scloud1.frcloud.io   <none>           <none>
rook-ceph-mon-a-75cd466dc8-hfbsg                                 1/1     Running     0          3h8m   10.100.94.39     k8scloud1.frcloud.io   <none>           <none>
rook-ceph-mon-b-657f47d7c7-4pgz5                                 1/1     Running     0          3h8m   10.100.177.28    k8scloud3.frcloud.io   <none>           <none>
rook-ceph-mon-c-56dd8ccff7-lspxw                                 1/1     Running     0          3h8m   10.100.144.27    k8scloud2.frcloud.io   <none>           <none>
rook-ceph-operator-7cbfbf4ddc-lmtbb                              1/1     Running     0          3h9m   10.100.177.22    k8scloud3.frcloud.io   <none>           <none>
rook-ceph-osd-0-7899ff86c4-ch4nx                                 1/1     Running     0          3h7m   10.100.177.31    k8scloud3.frcloud.io   <none>           <none>
rook-ceph-osd-1-7957fc4c89-kwmbr                                 1/1     Running     0          3h7m   10.100.144.30    k8scloud2.frcloud.io   <none>           <none>
rook-ceph-osd-2-76bcc5c95b-kblng                                 1/1     Running     0          3h7m   10.100.94.43     k8scloud1.frcloud.io   <none>           <none>
rook-ceph-osd-prepare-k8scloud1.frcloud.io-p9h45                 0/1     Completed   0          127m   10.100.94.47     k8scloud1.frcloud.io   <none>           <none>
rook-ceph-osd-prepare-k8scloud2.frcloud.io-8g9bh                 0/1     Completed   0          127m   10.100.144.35    k8scloud2.frcloud.io   <none>           <none>
rook-ceph-osd-prepare-k8scloud3.frcloud.io-sjt85                 0/1     Completed   0          127m   10.100.177.36    k8scloud3.frcloud.io   <none>           <none>
rook-discover-n2k9c                                              1/1     Running     0          3h9m   10.100.177.23    k8scloud3.frcloud.io   <none>           <none>
rook-discover-n2ppm                                              1/1     Running     0          3h9m   10.100.144.24    k8scloud2.frcloud.io   <none>           <none>
rook-discover-tgcqb                                              1/1     Running     0          3h9m   10.100.238.147   k8scloud5.frcloud.io   <none>           <none>
rook-discover-wcv9d                                              1/1     Running     0          3h9m   10.100.72.15     k8scloud4.frcloud.io   <none>           <none>
rook-discover-xt8pq                                              1/1     Running     0          3h9m   10.100.94.37     k8scloud1.frcloud.io   <none>           <none>

```




## 配置ceph dashboard


+ 看一眼dashboard在哪个service上
```bash
kubectl -n rook-ceph get service
#可以看到dashboard监听了8443端口
```

+ 创建个nodeport类型的service以便集群外部访问
```bash
kubectl apply -f dashboard-external-https.yaml
# 查看一下nodeport在哪个端口
ss -tanl
kubectl -n rook-ceph get service
```
+ 找出Dashboard的登陆账号和密码
```bash
MGR_POD=`kubectl get pod -n rook-ceph | grep mgr | awk '{print $1}'`
kubectl -n rook-ceph logs $MGR_POD | grep password
```

+ 打开浏览器输入任意一个Node的IP+nodeport端口

## 配置ceph RBD为storageclass

+ 官方提供的配置模板
```bash
/stage/rook/cluster/examples/kubernetes/ceph/csi/rbd/storageclass.yaml
```


+ 配置范例
```bash
apiVersion: ceph.rook.io/v1beta1
kind: Pool
metadata:
  #这个name就是创建成ceph pool之后的pool名字
  name: replicapool
  namespace: rook-ceph
spec:
  replicated:
    size: 1
  # size 池中数据的副本数,1就是不保存任何副本
  failureDomain: osd
  #  failureDomain：数据块的故障域，
  #  值为host时，每个数据块将放置在不同的主机上
  #  值为osd时，每个数据块将放置在不同的osd上
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: ceph
   # StorageClass的名字，pvc调用时填的名字
provisioner: ceph.rook.io/block
parameters:
  pool: replicapool
  # Specify the namespace of the rook cluster from which to create volumes.
  # If not specified, it will use `rook` as the default namespace of the cluster.
  # This is also the namespace where the cluster will be
  clusterNamespace: rook-ceph
  # Specify the filesystem type of the volume. If not specified, it will use `ext4`.
  fstype: xfs
# 设置回收策略默认为：Retain
reclaimPolicy: Retain


```

+ 创建storageclass
```bash
kubectl apply -f storageclass.yaml
[root@k8scloud1 ceph]# kubectl get storageclasses.storage.k8s.io  -n rook-ceph
NAME              PROVISIONER                  AGE
rook-ceph-block   rook-ceph.rbd.csi.ceph.com   47s
[root@k8scloud1 ceph]# kubectl describe storageclasses.storage.k8s.io  -n rook-ceph
Name:                  rook-ceph-block
IsDefaultClass:        No
Annotations:           <none>
Provisioner:           rook-ceph.rbd.csi.ceph.com
Parameters:            clusterID=rook-ceph,csi.storage.k8s.io/fstype=xfs,csi.storage.k8s.io/node-stage-secret-name=rook-csi-rbd-node,csi.storage.k8s.io/node-stage-secret-namespace=rook-ceph,csi.storage.k8s.io/provisioner-secret-name=rook-csi-rbd-provisioner,csi.storage.k8s.io/provisioner-secret-namespace=rook-ceph,imageFeatures=layering,imageFormat=2,pool=replicapool
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Retain
VolumeBindingMode:     Immediate
Events:                <none>
```

## 配置ceph FS为storageclass

+ 官方提供的配置模板
```bash
/stage/rook/cluster/examples/kubernetes/ceph/filesystem.yaml
/stage/rook/cluster/examples/kubernetes/ceph/csi/cephfs/storageclass.yaml
```


+ 配置范例
```bash
#filesystem.yaml
apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph
spec:
  metadataPool:
    replicated:
      size: 3
  dataPools:
    - replicated:
        size: 3
  preservePoolsOnDelete: true
  metadataServer:
    activeCount: 1
    activeStandby: true
    
    
#storageclass.yaml    
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-cephfs
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.cephfs.csi.ceph.com
parameters:
  # clusterID is the namespace where operator is deployed.
  clusterID: rook-ceph

  # CephFS filesystem name into which the volume shall be created
  fsName: myfs

  # Ceph pool into which the volume shall be created
  # Required for provisionVolume: "true"
  pool: myfs-data0

  # Root path of an existing CephFS volume
  # Required for provisionVolume: "false"
  # rootPath: /absolute/path

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph

reclaimPolicy: Retain
```
+ 创建文件系统

```bash
## Create the filesystem
$ kubectl create -f filesystem.yaml
[...]
## To confirm the filesystem is configured, wait for the mds pods to start
$ kubectl -n rook-ceph get pod -l app=rook-ceph-mds
NAME                                      READY     STATUS    RESTARTS   AGE
rook-ceph-mds-myfs-7d59fdfcf4-h8kw9       1/1       Running   0          12s
rook-ceph-mds-myfs-7d59fdfcf4-kgkjp       1/1       Running   0          12s

```

+ 创建storageclass
```bash
kubectl apply -f storageclass.yaml
[root@k8scloud1 ceph]# kubectl get storageclasses.storage.k8s.io  -n rook-ceph
NAME              PROVISIONER                     AGE
csi-cephfs        rook-ceph.cephfs.csi.ceph.com   17s
rook-ceph-block   rook-ceph.rbd.csi.ceph.com      17m
[root@k8scloud1 ceph]# kubectl describe storageclasses.storage.k8s.io  -n rook-ceph
Name:                  csi-cephfs
IsDefaultClass:        No
Annotations:           <none>
Provisioner:           rook-ceph.cephfs.csi.ceph.com
Parameters:            clusterID=rook-ceph,csi.storage.k8s.io/node-stage-secret-name=rook-csi-cephfs-node,csi.storage.k8s.io/node-stage-secret-namespace=rook-ceph,csi.storage.k8s.io/provisioner-secret-name=rook-csi-cephfs-provisioner,csi.storage.k8s.io/provisioner-secret-namespace=rook-ceph,fsName=myfs,pool=myfs-data0
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Retain
VolumeBindingMode:     Immediate
Events:                <none>


Name:                  rook-ceph-block
IsDefaultClass:        No
Annotations:           <none>
Provisioner:           rook-ceph.rbd.csi.ceph.com
Parameters:            clusterID=rook-ceph,csi.storage.k8s.io/fstype=xfs,csi.storage.k8s.io/node-stage-secret-name=rook-csi-rbd-node,csi.storage.k8s.io/node-stage-secret-namespace=rook-ceph,csi.storage.k8s.io/provisioner-secret-name=rook-csi-rbd-provisioner,csi.storage.k8s.io/provisioner-secret-namespace=rook-ceph,imageFeatures=layering,imageFormat=2,pool=replicapool
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Retain
VolumeBindingMode:     Immediate
Events:                <none>

```
+ 创建个nginx pod尝试挂载
```bash
cat << EOF > nginx.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-cephfs

---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports: 
  - port: 80
    name: nginx-port
    targetPort: 80
    protocol: TCP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /html
          name: http-file
      volumes:
      - name: http-file
        persistentVolumeClaim:
          claimName: nginx-pvc
EOF
```
kubectl apply -f nginx.yaml
## 清理rook

+ 删除Block and File artifacts
```bash
kubectl delete -f ../wordpress.yaml
kubectl delete -f ../mysql.yaml
kubectl delete -n rook-ceph cephblockpool replicapool
kubectl delete storageclass rook-ceph-block
kubectl delete -f csi/cephfs/kube-registry.yaml
kubectl delete storageclass rook-cephfs
```

+ 删除CephCluster CRD
```bash
kubectl -n rook-ceph delete cephcluster rook-ceph
```

+ 删除Operator and related Resources
```bash
cd cluster/examples/kubernetes/ceph
kubectl delete -f operator.yaml
kubectl delete -f common.yaml
```
+ 删除主机磁盘上的数据(重要)
```bash
DISK="/dev/vdb"
# Zap the disk to a fresh, usable state (zap-all is important, b/c MBR has to be clean)
# You will have to run this step for all disks.
yum install -y gdisk
sgdisk --zap-all $DISK

# These steps only have to be run once on each node
# If rook sets up osds using ceph-volume, teardown leaves some devices mapped that lock the disks.
ls /dev/mapper/ceph-* | xargs -I% -- dmsetup remove %
# ceph-volume setup can leave ceph-<UUID> directories in /dev (unnecessary clutter)
rm -rf /dev/ceph-*
#务必清除干净
rm -rf /va/lib/rook/*
dd if=/dev/zero of=/dev/vdb bs=1M count=10240 oflag=direct
```
