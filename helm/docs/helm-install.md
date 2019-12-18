
# helm 安装配置




## helm 客户端配置
```bash
cd /stage
curl -LO https://www.cnrancher.com/download/helm/helm-v2.14.3-linux-amd64.tar.gz
tar -xvf helm-v2.14.3-linux-amd64.tar.gz
sudo cp linux-amd64/helm /usr/local/bin/

[root@k8scloud1 stage]# helm version
Client: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
Error: could not find tiller

```
## helm 服务端配置

```bash 
#添加helm service account 并添加到clusteradmin 这个clusterrole上
# 创建账号、绑定角色
kubectl create serviceaccount --namespace=kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller







#碰到 Error: error installing: the server could not find the requested resource 的错误
#对于 Kubernetes v1.16.0 以上的版本，有可能会碰到 Error: error installing: the server could not find the requested resource 的错误。这是由于 extensions/v1beta1 已经被 apps/v1 替代。相信在2.15 或者 3 版本发布之后, 应该就不会遇到这个问题了。还是生态比较慢的原因。

#解决方法:
helm init --output yaml > tiller.yaml 
#使用命令:
helm init -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.14.3 --stable-repo-url http://mirror.azure.cn/kubernetes/charts/ --service-account tiller --override spec.selector.matchLabels.'name'='tiller',spec.selector.matchLabels.'app'='helm' --output yaml | sed 's@apiVersion: extensions/v1beta1@apiVersion: apps/v1@' | kubectl apply -f -

# 给tiller设置角色
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' deployment.extensions "tiller-deploy" patched

# 查看授权
kubectl get deploy --namespace kube-system   tiller-deploy  --output yaml|grep  serviceAccount
      serviceAccount: helm-tiller
      serviceAccountName: helm-tiller
      
      
替换 repo 为阿里镜像，注意上面的命令已经做了这一步，所以不用做了。这里是对尚未替换repo的情况下，如何替换repo的介绍。
root@rancherk8sm1:~# helm repo list
NAME    URL
stable  https://kubernetes-charts.storage.googleapis.com
local   http://127.0.0.1:8879/charts



root@rancherk8sm1:~# helm repo remove stable
"stable" has been removed from your repositories

root@rancherk8sm1:~# helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
"stable" has been added to your repositories
root@rancherk8sm1:~# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈



```


如果这里的权限问题没有处理好，安装完成后会发现命令没有权限，
root@master:/home/hzz# helm ls
Error: configmaps is forbidden: User "system:serviceaccount:kube-system:default" cannot list resource "configmaps" in API group "" in the namespace "kube-system"

可以通过helm reset -f或者helm reset --force强制删除tiller容器后，使用正确的参数重新进行helm init操作
3、验证安装

出现helm client 和server之后helm便安装完成
root@master:/home/hzz# kubectl -n kube-system get pods|grep tiller-deploy
tiller-deploy-7d49974877-w78nz          1/1     Running   0          4h46m

root@master:/home/hzz# helm version
Client: &version.Version{SemVer:"v2.14.0", GitCommit:"05811b84a3f93603dd6c2fcfe57944dfa7ab7fd0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.0", GitCommit:"05811b84a3f93603dd6c2fcfe57944dfa7ab7fd0", GitTreeState:"clean"}

可执行helm list查看K8S中已安装的charts 。
二、使用
1、更换仓库
# 先移除原先的仓库
root@master:/home/hzz# helm repo remove stable
"stable" has been removed from your repositories

# 添加新的仓库地址
root@master:/home/hzz# helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
"stable" has been added to your repositories

# 更新仓库
root@master:/home/hzz# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete.

2、查看可获取的Chart
root@master:/home/hzz# helm search
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                     
stable/acs-engine-autoscaler    2.1.3           2.1.1           Scales worker nodes within agent pools                      
stable/aerospike                0.1.7           v3.14.1.2       A Helm chart for Aerospike in Kubernetes                    
stable/anchore-engine           0.1.3           0.1.6           Anchore container analysis and policy evaluation engine s...
stable/artifactory              7.0.3           5.8.4           Universal Repository Manager supporting all major packagi...
stable/artifactory-ha           0.1.0           5.8.4           Universal Repository Manager supporting all major packagi...
stable/aws-cluster-autoscaler   0.3.2                           Scales worker nodes within autoscaling groups.              
stable/bitcoind                 0.1.0           0.15.1          Bitcoin is an innovative payment network and a new kind o...
..................



## 如何初始化





