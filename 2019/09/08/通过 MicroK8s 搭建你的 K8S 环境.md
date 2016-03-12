# 通过 MicroK8s 搭建你的 K8s 环境

去年的时候，我曾经写过如何[简单搭建 Kubernetes 集群](https://soulteary.com/2018/10/03/how-to-get-your-k8s-cluster.html)，当时使用的是官方的工具箱：Kubeadm，这个方案对于只是想试试的同学来说，还是过于复杂。这里介绍一款简单的工具：MicroK8s。

官方给这款工具的人设是“无需运维的 Kubernetes ，服务于工作站、物联网。”最大的价值在于可以快速搭建单节点的容器编排系统，用于生产试验。

[官方网站](https://microk8s.io/)里的文档有简单介绍如何安装使用，但是却未曾考虑安装过程存在网络问题的神州大陆的同学们，本文将结合这种情况聊聊。

## 写在前面

官方在[早些时候](https://discuss.kubernetes.io/t/containerd-and-security-updates-on-the-next-microk8s-release/4844)宣布接下来将会使用 `Containerd` 替换 `docker`：

> The upcoming release of v1.14 Kubernetes will mark the MicroK8s switch to Containerd and enhanced security. As this is a big step forward we would like to give you a heads up and offer you a preview of what is coming. Give it a test drive with:
> 
> snap install microk8s --classic --channel=1.13/edge/secure-containerd
> 
> You can read more in our blog 117, and the respective pill request 13
> Please, let us know 5 how we can make this transition smoother for you.
> Thanks

社区里已经有用户[咨询/吐槽过了](https://github.com/ubuntu/microk8s/issues/382)，这里考虑减少变化，暂时还是以使用 docker 作为容器封装的 1.13 ，新版本留给下一篇“折腾”吧。

## 使用 SNAP 安装 MicroK8S

**snap** 是 **canonical ** 公司给出的更“高级”的包管理的解决方案，最早应用在 Ubuntu Phone 上。

使用 snap 安装 K8s 确实很简单，就像下面一样，一条命令解决问题：

```bash
snap install microk8s --classic --channel=1.13/stable
```

但是这条命令如果不是在海外主机上执行，应该会遇到安装缓慢的问题。

```bash
snap install microk8s --classic --channel=1.13/stable
Download snap "microk8s" (581) from channel "1.13/stable"                                                                                            0% 25.9kB/s 2h32m
```

想要解决这个问题，暂时只能给 snap 添加代理来解决问题，snap 不会读取系统的环境变量，只读取应用的变量文件。

使用下面的命令可以方便的修改 snap 的环境变量，但是默认编辑器是 ** nano **，非常难用。

```bash
systemctl edit snapd.service
```

这里可以先更新编辑器为我们熟悉的 ** vim **：

```bash
sudo update-alternatives --install "$(which editor)" editor "$(which vim)" 15
sudo update-alternatives --config editor
```

交互式终端需要我们手动输入数字，然后按下回车确认选择：

```bash
There are 5 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /bin/nano            40        auto mode
  1            /bin/ed             -100       manual mode
  2            /bin/nano            40        manual mode
  3            /usr/bin/vim         15        manual mode
  4            /usr/bin/vim.basic   30        manual mode
  5            /usr/bin/vim.tiny    15        manual mode

Press <enter> to keep the current choice[*], or type selection number: 5
update-alternatives: using /usr/bin/vim.tiny to provide /usr/bin/editor (editor) in manual mode
```

再次执行上面编辑环境变量的命令，添加一段代理配置：

```bash
[Service]

Environment="HTTP_PROXY=http://10.11.12.123:10240"
Environment="HTTPS_PROXY=http://10.11.12.123:10240"
Environment="NO_PROXY=localhost,127.0.0.1,192.168.0.0/24,*.domain.ltd"
```

再次执行安装，安装进度起飞：

```TeXT
snap install microk8s --classic --channel=1.13/stable

Download snap "microk8s" (581) from channel "1.13/stable"                                                                                           31% 14.6MB/s 11.2s
```

如果速度没有变化，可以考虑重载 snap  服务。

```bash
systemctl daemon-reload && systemctl restart snapd
```

如果上面的操作一切顺利，你将会看到类似下面的日志：

```TeXT
snap install microk8s --classic --channel=1.13/stable
microk8s (1.13/stable) v1.13.6 from Canonical✓ installed
```

执行列表命令，可以看到当前 snap 已经安装好的工具：

```bash
snap list
Name      Version  Rev   Tracking  Publisher   Notes
core      16-2.40  7396  stable    canonical✓  core
microk8s  v1.13.6  581   1.13      canonical✓  classic
```

之前 **独立安装 K8s 需要先安装 docker**，而使用 snap 安装的话，这一切都是默认就绪的。

```bash
microk8s.docker version
Client:
 Version:           18.09.2
 API version:       1.39
 Go version:        go1.10.4
 Git commit:        6247962
 Built:             Tue Feb 26 23:56:24 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.09.2
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.4
  Git commit:       6247962
  Built:            Tue Feb 12 22:47:29 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```

## 获取 Kubernetes 依赖镜像

想要使用 Kubernetes，除了安装 MicroK8s 外，还需要获取它依赖的工具镜像。然而镜像的获取还是需要费点功夫的，首先获取 `1.13` 版本的 MicroK8s 代码：

```bash
git clone --single-branch --branch=1.13 https://github.com/ubuntu/microk8s.git
```

然后获取其中声明的容器镜像列表：

```bash
grep -ir 'image:' * | awk '{print $3 $4}' | uniq
```

因为官方代码的奔放，我们会获得长得奇形怪状的镜像名称：

```TeXT
localhost:32000/my-busybox
elasticsearch:6.5.1
alpine:3.6
docker.elastic.co/kibana/kibana-oss:6.3.2
time="2016-02-04T07:53:57.505612354Z"level=error
cdkbot/registry-$ARCH:2.6
...
...
quay.io/prometheus/prometheus
quay.io/coreos/kube-rbac-proxy:v0.4.0
k8s.gcr.io/metrics-server-$ARCH:v0.2.1
cdkbot/addon-resizer-$ARCH:1.8.1
cdkbot/microbot-$ARCH
"k8s.gcr.io/cuda-vector-add:v0.1"
nginx:latest
istio/examples-bookinfo-details-v1:1.8.0
busybox
busybox:1.28.4
```

根据我们要部署的目标服务器的具体需求，替换掉 `$ARCH` 变量，去掉无意义的镜像名称，整理好的列表如下：

```TeXT
k8s.gcr.io/fluentd-elasticsearch:v2.2.0
elasticsearch:6.5.1
alpine:3.6
docker.elastic.co/kibana/kibana-oss:6.3.2
cdkbot/registry-amd64:2.6
gcr.io/google_containers/defaultbackend-amd64:1.4
quay.io/kubernetes-ingress-controller/nginx-ingress-controller-amd64:0.22.0
jaegertracing/jaeger-operator:1.8.1
gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.7
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.7
gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.7
k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.3
k8s.gcr.io/heapster-influxdb-amd64:v1.3.3
k8s.gcr.io/heapster-grafana-amd64:v4.4.3
k8s.gcr.io/heapster-amd64:v1.5.2
cdkbot/addon-resizer-amd64:1.8.1
cdkbot/hostpath-provisioner-amd64:latest
quay.io/coreos/k8s-prometheus-adapter-amd64:v0.3.0
grafana/grafana:5.2.4
quay.io/coreos/kube-rbac-proxy:v0.4.0
quay.io/coreos/kube-state-metrics:v1.4.0
quay.io/coreos/addon-resizer:1.0
quay.io/prometheus/prometheus
quay.io/coreos/prometheus-operator:v0.25.0
quay.io/prometheus/alertmanager
quay.io/prometheus/node-exporter:v0.16.0
quay.io/coreos/kube-rbac-proxy:v0.4.0
k8s.gcr.io/metrics-server-amd64:v0.2.1
cdkbot/addon-resizer-amd64:1.8.1
nvidia/k8s-device-plugin:1.11
cdkbot/microbot-amd64
k8s.gcr.io/cuda-vector-add:v0.1
nginx:latest
istio/examples-bookinfo-details-v1:1.8.0
istio/examples-bookinfo-ratings-v1:1.8.0
istio/examples-bookinfo-reviews-v1:1.8.0
istio/examples-bookinfo-reviews-v2:1.8.0
istio/examples-bookinfo-reviews-v3:1.8.0
istio/examples-bookinfo-productpage-v1:1.8.0
busybox
busybox:1.28.4
```

将上面的列表保存为 **package-list.txt** 在网络通畅的云服务器上使用下面的脚本，可以将 K8s 依赖的工具镜像离线保存：

```bash
PACKAGES=`cat ./package-list.txt`;

for package in $PACKAGES; do docker pull "$package"; done

docker images | tail -n +2 | grep -v "<none>" | awk '{printf("%s:%s\n", $1, $2)}' | while read IMAGE; do
    for package in $PACKAGES;
    do
        if [[ $package != *[':']* ]];then package="$package:latest"; fi

        if [ $IMAGE == $package ];then
            echo "[find image] $IMAGE"
            filename="$(echo $IMAGE| tr ':' '-' | tr '/' '-').tar"
            echo "[save as] $filename"
            docker save ${IMAGE} -o $filename
        fi
    done
done
```

将镜像转存待部署服务器的方式多种多样，这里提一种最简单的方案：`scp`

```bash
PACKAGES=`cat ./package-list.txt`;

for package in $PACKAGES;
do
    if [[ $package != *[':']* ]];then package="$package:latest";fi
    filename="$(echo $package| tr ':' '-' | tr '/' '-').tar"
    # 根据自己实际场景修改地址
    scp "mirror-server:~/images/$filename" .
    scp "./$filename" "deploy-server:"
done
```

如果顺利你将看到类似下面的日志：

```TeXT
k8s.gcr.io-fluentd-elasticsearch-v2.2.0.tar                                                                                         100%  140MB  18.6MB/s   00:07
elasticsearch-6.5.1.tar                                                                                                             100%  748MB  19.4MB/s   00:38
alpine-3.6.tar                                                                                                                      100% 4192KB  15.1MB/s   00:00
docker.elastic.co-kibana-kibana-oss-6.3.2.tar                                                                                       100%  614MB  22.8MB/s   00:26
cdkbot-registry-amd64-2.6.tar                                                                                                       100%  144MB  16.1MB/s   00:08
gcr.io-google_containers-defaultbackend-amd64-1.4.tar                                                                               100% 4742KB  13.3MB/s   00:00
...
...
```

最后在目标服务器使用 `docker load` 命令导入镜像即可。

```bash
ls *.tar | xargs -I {} microk8s.docker load -i {}
```

接下来可以正式安装 K8s 啦。

## 正式开始安装 Kubernetes

使用 MicroK8s 配置各种组件很简单，只需要一条命令：

```bash
microk8s.enable dashboard dns ingress istio registry storage
```

完整的组件列表可以通过 `microk8s.enable --help` 来查看：

```TeXT
microk8s.enable --help
Usage: microk8s.enable ADDON...
Enable one or more ADDON included with microk8s
Example: microk8s.enable dns storage

Available addons:

  dashboard
  dns
  fluentd
  gpu
  ingress
  istio
  jaeger
  metrics-server
  prometheus
  registry
  storage
```

执行 `enable` 顺利的话，你将看到类似下面的日志：

```TeXT
logentry.config.istio.io/accesslog created
logentry.config.istio.io/tcpaccesslog created
rule.config.istio.io/stdio created
rule.config.istio.io/stdiotcp created
...
...
Istio is starting
Enabling the private registry
Enabling default storage class
deployment.extensions/hostpath-provisioner created
storageclass.storage.k8s.io/microk8s-hostpath created
Storage will be available soon
Applying registry manifest
namespace/container-registry created
persistentvolumeclaim/registry-claim created
deployment.extensions/registry created
service/registry created
The registry is enabled
Enabling default storage class
deployment.extensions/hostpath-provisioner unchanged
storageclass.storage.k8s.io/microk8s-hostpath unchanged
Storage will be available soon
```

使用 `microk8s.status` 检查各个组件的状态：

```TeXT
microk8s is running
addons:
jaeger: disabled
fluentd: disabled
gpu: disabled
storage: enabled
registry: enabled
ingress: enabled
dns: enabled
metrics-server: disabled
prometheus: disabled
istio: enabled
dashboard: enabled
```

但是组件就绪，不代表 K8s 已经安装就绪，使用 `microk8s.inspect`  排查下安装部署结果：

```TeXT
Inspecting services
  Service snap.microk8s.daemon-containerd is running
  Service snap.microk8s.daemon-docker is running
  Service snap.microk8s.daemon-apiserver is running
  Service snap.microk8s.daemon-proxy is running
  Service snap.microk8s.daemon-kubelet is running
  Service snap.microk8s.daemon-scheduler is running
  Service snap.microk8s.daemon-controller-manager is running
  Service snap.microk8s.daemon-etcd is running
  Copy service arguments to the final report tarball
Inspecting AppArmor configuration
Gathering system info
  Copy network configuration to the final report tarball
  Copy processes list to the final report tarball
  Copy snap list to the final report tarball
  Inspect kubernetes cluster

 WARNING:  IPtables FORWARD policy is DROP. Consider enabling traffic forwarding with: sudo iptables -P FORWARD ACCEPT
```

解决方法很简单，使用 `ufw` 添加几条规则即可：

```bash
sudo ufw allow in on cbr0 && sudo ufw allow out on cbr0
sudo ufw default allow routed
sudo iptables -P FORWARD ACCEPT
```

再次使用 `microk8s.inspect` 命令检查，会发现 **WARNING** 已经消失了。

但是 Kubernetes 真的安装就绪了吗？跟随下一小节寻找答案吧。

## 解决 Kubernetes 不能正常启动

在上面的操作顺利之后完毕后，使用 `microk8s.kubectl get pods` 查看当前 Kubernetes pods 状态，如果看到 `ContainerCreating` ，那么说明 Kubernetes 还需要一些额外的“修补工作”。

```TeXT
NAME                                      READY   STATUS              RESTARTS   AGE
default-http-backend-855bc7bc45-t4st8     0/1     ContainerCreating   0          16m
nginx-ingress-microk8s-controller-kgjtl   0/1     ContainerCreating   0          16m
```

使用 `microk8s.kubectl get pods --all-namespaces` 查看详细的状态，不出意外的话，将看到类似下面的日志输出：

```TeXT
NAMESPACE            NAME                                              READY   STATUS              RESTARTS   AGE
container-registry   registry-7fc4594d64-rrgs9                         0/1     Pending             0          15m
default              default-http-backend-855bc7bc45-t4st8             0/1     ContainerCreating   0          16m
default              nginx-ingress-microk8s-controller-kgjtl           0/1     ContainerCreating   0          16m
...
...
```

首要的问题就是解决掉这个处于 **Pending** 状态的容器。使用 `microk8s.kubectl describe pod` 可以快速查看当前这个问题 pod 的详细状态：

```TeXT
Events:
  Type     Reason                  Age                 From                         Message
  ----     ------                  ----                ----                         -------
  Normal   Scheduled               22m                 default-scheduler            Successfully assigned default/default-http-backend-855bc7bc45-t4st8 to ubuntu-basic-18-04
  Warning  FailedCreatePodSandBox  21m                 kubelet, ubuntu-basic-18-04  Failed create pod sandbox: rpc error: code = Unknown desc = failed pulling image "k8s.gcr.io/pause:3.1": Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
  Warning  FailedCreatePodSandBox  43s (x45 over 21m)  kubelet, ubuntu-basic-18-04  Failed create pod sandbox: rpc error: code = Unknown desc = failed pulling image "k8s.gcr.io/pause:3.1": Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

参考日志输出，可以发现，之前整理的依赖镜像列表，还存在“漏网之鱼”： MicroK8s 中还包含未写在程序中，从远程配置中获取的镜像。

对于这种情况，我们只能通过给 `docker` 添加代理来解决问题（或者手动一个一个来）。

编辑 MicroK8s 使用的 docker 环境变量配置文件 `vi /var/snap/microk8s/current/args/dockerd-env`，在其中添加代理配置，比如：

```bash
HTTPS_PROXY=http://10.11.12.123:10555
HTTPS_PROXY=http://10.11.12.123:10555
NO_PROXY=127.0.0.1
```

接着重启 docker ：

```bash
sudo systemctl restart snap.microk8s.daemon-docker.service
```

这一切就绪之后，执行下面的命令，重置 MicroK8s 并再次尝试安装各种组件：

```bash
microk8s.reset
microk8s.enable dashboard dns ingress istio registry storage
```

命令之后完毕之后，片刻之后再次执行 `microk8s.kubectl get pods` 会发现所有的 pod 的状态就都是 Running：

```TeXT
NAME                                      READY   STATUS    RESTARTS   AGE
default-http-backend-855bc7bc45-w62jd     1/1     Running   0          46s
nginx-ingress-microk8s-controller-m9lc2   1/1     Running   0          46s
```

使用 `microk8s.kubectl get pods --all-namespaces` 继续进行验证：

```TeXT

NAMESPACE            NAME                                              READY   STATUS      RESTARTS   AGE
container-registry   registry-7fc4594d64-whjnl                         1/1     Running     0          2m
default              default-http-backend-855bc7bc45-w62jd             1/1     Running     0          2m
default              nginx-ingress-microk8s-controller-m9lc2           1/1     Running     0          2m
istio-system         grafana-59b8896965-xtc27                          1/1     Running     0          2m
istio-system         istio-citadel-856f994c58-fbc7c                    1/1     Running     0          2m
istio-system         istio-cleanup-secrets-9q8tw                       0/1     Completed   0          2m
istio-system         istio-egressgateway-5649fcf57-cbqlv               1/1     Running     0          2m
istio-system         istio-galley-7665f65c9c-l7grc                     1/1     Running     0          2m
istio-system         istio-grafana-post-install-sl6mb                  0/1     Completed   0          2m
istio-system         istio-ingressgateway-6755b9bbf6-hvnld             1/1     Running     0          2m
istio-system         istio-pilot-698959c67b-zts2v                      2/2     Running     0          2m
istio-system         istio-policy-6fcb6d655f-mx68m                     2/2     Running     0          2m
istio-system         istio-security-post-install-5d7bb                 0/1     Completed   0          2m
istio-system         istio-sidecar-injector-768c79f7bf-qvcjd           1/1     Running     0          2m
istio-system         istio-telemetry-664d896cf5-jz22s                  2/2     Running     0          2m
istio-system         istio-tracing-6b994895fd-z8jn9                    1/1     Running     0          2m
istio-system         prometheus-76b7745b64-fqvn9                       1/1     Running     0          2m
istio-system         servicegraph-5c4485945b-spf77                     1/1     Running     0          2m
kube-system          heapster-v1.5.2-64874f6bc6-8ghnr                  4/4     Running     0          2m
kube-system          hostpath-provisioner-599db8d5fb-kxtjw             1/1     Running     0          2m
kube-system          kube-dns-6ccd496668-98mvt                         3/3     Running     0          2m
kube-system          kubernetes-dashboard-654cfb4879-vzgk5             1/1     Running     0          2m
kube-system          monitoring-influxdb-grafana-v4-6679c46745-68vn7   2/2     Running     0          2m
```

如果你看到的结果类似上面这样，说明 Kubernetes 是真的就绪了。

## 快速创建应用

安都安完了，总得试着玩玩看吧，当然，这里不会随大流的展示下管理后台就匆匆搁笔。

使用 `kubectl` 基于现成的容器创建一个 deployment：

```bash
microk8s.kubectl create deployment microbot --image=dontrebootme/microbot:v1
```

既然用上了最先进的编排系统，不体验下自动扩容岂不是太可惜了：

```bash
microk8s.kubectl scale deployment microbot --replicas=2
```

将服务暴露出来，创建流量转发：

```bash
microk8s.kubectl expose deployment microbot --type=NodePort --port=80 --name=microbot-service
```

使用 `get` 命令查看服务状态：

```bash
microk8s.kubectl get all
```

如果一切顺利的话，你将会看到类似下面的日志输出：

```TeXT
NAME                                          READY   STATUS    RESTARTS   AGE
pod/default-http-backend-855bc7bc45-w62jd     1/1     Running   0          64m
pod/microbot-7c7594fb4-dxgg7                  1/1     Running   0          13m
pod/microbot-7c7594fb4-v9ztg                  1/1     Running   0          13m
pod/nginx-ingress-microk8s-controller-m9lc2   1/1     Running   0          64m

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/default-http-backend   ClusterIP   10.152.183.13   <none>        80/TCP         64m
service/kubernetes             ClusterIP   10.152.183.1    <none>        443/TCP        68m
service/microbot-service       NodePort    10.152.183.15   <none>        80:31354/TCP   13m

NAME                                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/nginx-ingress-microk8s-controller   1         1         1       1            1           <none>          64m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/default-http-backend   1/1     1            1           64m
deployment.apps/microbot               2/2     2            2           13m

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/default-http-backend-855bc7bc45   1         1         1       64m
replicaset.apps/microbot-7c7594fb4                2         2         2       13m
```

可以看到我们刚刚创建的 Service 地址是 `10.11.12.234:31354`。使用浏览器访问，可以看到应用已经跑起来啦。

![刚刚就绪的应用](https://attachment.soulteary.com/2019/09/08/ready.png)

本着“谁制造谁收拾”的绿色环保理念，除了“无脑”创建外，我们也需要学会如何治理（销毁），使用 `delete` 命令，先销毁 deployment ：

```bash
microk8s.kubectl delete deployment microbot
```

执行完毕后日志输出会是下面一样：

```TeXT
deployment.extensions "microbot" deleted
```

在销毁 service 前，我们需要使用 `get` 命令先获取所有的 service 的名称：

```bash
microk8s.kubectl get services microbot-service

NAME               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
microbot-service   NodePort   10.152.183.15   <none>        80:31354/TCP   24m
```

得到了 service 名称后，依旧是使用 `delete` 命令，删除不需要的资源：

```bash
microk8s.kubectl delete service microbot-service
```

执行结果如下：

```TeXT
service "microbot-service" deleted
```

## 查看 Dashboard

估计还是有同学会想一窥 Dashboard 的状况。

可以通过 `microk8s.config` 命令，先获得当前服务器监听的 IP 地址：

```TeXT
microk8s.config
apiVersion: v1
clusters:
- cluster:
    server: http://10.11.12.234:8080
  name: microk8s-cluster
contexts:
- context:
    cluster: microk8s-cluster
    user: admin
  name: microk8s
current-context: microk8s
kind: Config
preferences: {}
users:
- name: admin
  user:
    username: admin
```

可以看到，当前监听的服务 IP 地址为 `10.11.12.234`，使用 `proxy` 命令，打开流量转发：

```bash
microk8s.kubectl proxy --accept-hosts=.* --address=0.0.0.0
```

接着访问下面的地址，就能看到我们熟悉的 Dashboard 啦：

```TeXT
http://10.11.12.234:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login
```

![熟悉的登陆界面](https://attachment.soulteary.com/2019/09/08/dashboard.png)

## 其他

完整安装下来，连着系统一共花费了接近 8G 的储存空间，所以如果你打算持续使用的话，可以提前规划下磁盘空间，比如参考 [ 迁移 Docker 容器储存位置 ](https://soulteary.com/2019/07/14/migrate-docker-container-storage-location.html) 把未来持续膨胀的 docker 镜像包地址先做个迁移。

```TeXT
df -h

Filesystem      Size  Used Avail Use% Mounted on
udev            7.8G     0  7.8G   0% /dev
tmpfs           1.6G  1.7M  1.6G   1% /run
/dev/sda2        79G   16G   59G  21% /
```

## 最后

这篇文章成文于一个月之前，由于使用的还是 “Docker” 方案，理论来说时效性还是靠谱的，如果你遇到了什么问题，欢迎讨论沟通。

看着草稿箱堆积越来越多的有趣内容，或许应该考虑“合作撰写”的模式了。

—EOF