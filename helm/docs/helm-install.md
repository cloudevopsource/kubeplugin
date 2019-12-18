
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


helm init \
    --history-max=3 \
    --tiller-image=gcr.azk8s.cn/kubernetes-helm/tiller:v2.14.3 \
    --stable-repo-url=https://mirror.azure.cn/kubernetes/charts/ \
    --service-account=helm-tiller
