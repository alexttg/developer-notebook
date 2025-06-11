# 使用minikube安装Kubernetes
> minikube 主要基于运行一个单节点 Kubernetes 集群，以便支持在本地机器上的 VM 内进行开发。它支持虚拟机驱动程序，如 VirtualBox、HyperV、KVM2。由于 Minikube 是 Kubernetes 体系中相对成熟的解决方案，支持的功能列表非常令人印象深刻。这些功能是负载均衡器、多集群、节点端口、持久卷、入口、仪表板或容器运行时。 

### 使用brew命令安装:
    brew install minikube
### 查看安装路径 
    which minikube 

`/opt/homebrew/bin/minikube`
### 启动 minikube
```
dengqing@mayundeMacBook-Pro ~ % minikube start
😄  Darwin 11.2 (arm64) 上的 minikube v1.26.1
👎  Unable to pick a default driver. Here is what was considered, in preference order:
💡  Alternatively you could install one of these drivers:
    ▪ docker: Not installed: exec: "docker": executable file not found in $PATH
    ▪ hyperkit: Not installed: exec: "hyperkit": executable file not found in $PATH
    ▪ parallels: Not installed: exec: "prlctl": executable file not found in $PATH
    ▪ vmware: Not installed: exec: "docker-machine-driver-vmware": executable file not found in $PATH
    ▪ virtualbox: Not installed: unable to find VBoxManage in $PATH
    ▪ podman: Not installed: exec: "podman": executable file not found in $PATH
    ▪ qemu2: Not installed: exec: "qemu-system-aarch64": executable file not found in $PATH

❌  Exiting due to DRV_NOT_DETECTED: No possible driver was detected. Try specifying --driver, or see https://minikube.sigs.k8s.io/docs/start/

```
原因是没有默认的驱动，安装docker 作为默认驱动

    brew install --cask --appdir=/Applications docker
启动成功

```
hk@mayundeMacBook-Pro ~ % minikube start
😄  Darwin 11.2 (arm64) 上的 minikube v1.26.1
🎉  minikube 1.29.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.29.0
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

✨  自动选择 docker 驱动
📌  Using Docker Desktop driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.24.3 preload ...
    > preloaded-images-k8s-v18-v1...:  342.82 MiB / 342.82 MiB  100.00% 924.64 
    > gcr.io/k8s-minikube/kicbase:  348.00 MiB / 348.00 MiB  100.00% 511.18 KiB
🔥  Creating docker container (CPUs=2, Memory=4000MB) ...
🐳  正在 Docker 20.10.17 中准备 Kubernetes v1.24.3…
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default 
```

打开Kubernetes控制台 ，出现以下错误

```
minikube dashboard
🔌  正在开启 dashboard ...
▪ Using image kubernetesui/dashboard:v2.6.0
▪ Using image kubernetesui/metrics-scraper:v1.0.8
🤔  正在验证 dashboard 运行情况 ...
🚀  Launching proxy ...
🤔  正在验证 proxy 运行状况 ...

❌  Exiting due to SVC_URL_TIMEOUT: http://127.0.0.1:5077/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ is not accessible: checkURL: Get "http://127.0.0.1:5077/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/": dial tcp 127.0.0.1:5077: connect: connection refused
```

### 删除minikube 重新启动并查看log输出

```
minikube delete
minikube start
minikube dashboard --alsologtostderr -v=
```
### 启动成功输出

```java
W0317 15:39:06.172493   56978 out.go:239] 🤔  正在验证 proxy 运行状况 ...
🤔  正在验证 proxy 运行状况 ...
I0317 15:39:06.188821   56978 dashboard.go:212] http://127.0.0.1:58992/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ response: <nil> &{Status:200 OK StatusCode:200 Proto:HTTP/1.1 ProtoMajor:1 ProtoMinor:1 Header:map[Accept-Ranges:[bytes] Audit-Id:[fe8d4733-509f-4140-b066-240ca91d76ef] Cache-Control:[no-cache, private no-cache, no-store, must-revalidate] Content-Type:[text/html; charset=utf-8] Date:[Fri, 17 Mar 2023 07:39:06 GMT] Last-Modified:[Tue, 31 May 2022 15:30:52 GMT]] Body:0x14000a5eae0 ContentLength:-1 TransferEncoding:[] Close:false Uncompressed:true Trailer:map[] Request:0x14000fa2c00 TLS:<nil>}
I0317 15:39:06.206370   56978 out.go:177] 🎉  Opening http://127.0.0.1:58992/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
🎉  Opening http://127.0.0.1:58992/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
I0317 15:39:06.415533   56978 dashboard.go:127] Success! I will now quietly sit around until kubectl proxy exits!
```

### 打开浏览器

[http://127.0.0.1:58992/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default](http://127.0.0.1:58992/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default)
![](https://cdn.nlark.com/yuque/0/2023/png/22527471/1679042313417-2fd59ab3-efa0-49a8-aac1-e6d42a096d80.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_67%2Ctext_eXVxdWUudHo%3D%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500%2Climit_0)
### 创建应用
在容器中尝试创建一个nginx

    kubectl create deployment test-nginx --image=nginx:1.23.0
暴露80端口

    kubectl expose deployment test-nginx --port=80  --type=NodePor
可以把 Node 主机端口 转发 到 pod 端口

    kubectl port-forward  --address 0.0.0.0   service/test-nginx 80:80
访问niginx  `http://localhost/`
![](https://cdn.nlark.com/yuque/0/2023/png/22527471/1679041514390-16a026a1-b9b9-47a1-9a51-acdb82c9b37b.png?x-oss-process=image%2Fresize%2Cw_1500%2Climit_0)
