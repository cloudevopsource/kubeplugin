
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
## 如何初始化





