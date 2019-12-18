
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
添加helm service account 并添加到clusteradmin 这个clusterrole上

kubectl create serviceaccount --namespace=kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

作者：陈Sir的知识库
链接：https://www.jianshu.com/p/7ab38da8758e
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

helm init \
    --history-max=3 \
    --tiller-image=registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.14.3 \
    --stable-repo-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts \
    --service-account=helm-tiller
    
    ://127.0.0.1:8879/charts 
