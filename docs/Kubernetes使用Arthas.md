#Kubernetes使用Arthas
###首先先安装kubectl

[下载地址](https://kubernetes.io/docs/tasks/tools/?spm=5176.2020520152.0.0.67b616dd7IeEDR)


将以下内容复制到计算机 $HOME/.kube/config 文件下（阿里云容器内环境凭证，容器服务-连接信息）

![](https://foruda.gitee.com/images/1685611751724591376/dc94a350_5094274.png)

通过 ` kubectl get pod `获取端口，选择服务器，`kubectl exec -it pod bash ` 启动服务器

###容器内安装并启动arthas

```
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jarcurl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar
```

切换k8s环境变量

	export KUBECONFIG=$KUBECONFIG:$HOME/.kube/config
查看k8s配置

	Kubectl config view