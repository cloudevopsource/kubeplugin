
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
kubectl create serviceaccount --namespace kube-system helm-tiller
kubectl create clusterrolebinding helm-tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:helm-tiller

# 给tiller设置角色
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"helm-tiller"}}}}' deployment.extensions "tiller-deploy" patched

# 查看授权
kubectl get deploy --namespace kube-system   tiller-deploy  --output yaml|grep  serviceAccount
      serviceAccount: helm-tiller
      serviceAccountName: helm-tiller

helm init \
    --history-max=3 \
    --tiller-image=registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.14.3 \
    --stable-repo-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts \
    --service-account=helm-tiller
    
    ://127.0.0.1:8879/charts 