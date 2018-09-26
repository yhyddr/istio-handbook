# 快速开始

在使用 Istio 前还是希望您有容器和 Kubernetes 的基础知识，如果您想要从零开始，那么可以使用 [**kubernetes-vagrant-centos-cluster**](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster) 并运行 [Bookinfo 应用](https://istio.io/zh/docs/examples/bookinfo/)来快速体验服务网格。

## 准备环境

需要准备以下软件和环境：

- 8G以上内存
- Vagrant 2.0+
- VirtualBox 5.0 +
- 提前下载Kubernetes 1.9以上版本（支持最新的1.11.0）的release压缩包
- Mac/Linux，**不支持Windows**

## 集群

我们使用 Vagrant 和 Virtualbox 安装包含 3 个节点的 Kubernetes 集群，其中 master 节点同时作为 node 节点。

| IP           | 主机名 | 组件                                                         |
| ------------ | ------ | ------------------------------------------------------------ |
| 172.17.8.101 | node1  | kube-apiserver、kube-controller-manager、kube-scheduler、etcd、kubelet、docker、flannel、dashboard |
| 172.17.8.102 | node2  | kubelet、docker、flannel、traefik                            |
| 172.17.8.103 | node3  | kubelet、docker、flannel                                     |

**注意**：以上的IP、主机名和组件都是固定在这些节点的，即使销毁后下次使用vagrant重建依然保持不变。

容器 IP 地址范围：172.33.0.0/30

Kubernetes service IP 地址范围：10.254.0.0/16

## 安装的组件

安装完成后的集群包含以下组件：

- flannel（`host-gw`模式）
- etcd（单节点）
- kubectl
- CoreDNS
- Kubernetes（版本根据下载的 kubernetes 安装包而定，支持 Kubernetes 1.9+）

- Istio

**可选插件**

- Kubernetes dashboard
- Helm
- Vistio
- Kiali

## 使用说明

将该repo克隆到本地，下载Kubernetes的到项目的根目录。

```bash
git clone https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster.git
cd kubernetes-vagrant-centos-cluster
wget https://storage.googleapis.com/kubernetes-release/release/v1.11.0/kubernetes-server-linux-amd64.tar.gz
```

注：您可以在[这里](https://kubernetes.io/docs/imported/release/notes/)找到Kubernetes的发行版下载地址。

使用vagrant启动集群。

```bash
vagrant up
```

如果是首次部署，会自动下载`centos/7`的box，这需要花费一些时间，另外每个节点还需要下载安装一系列软件包，整个过程大概需要10几分钟。

如果您在运行`vagrant up`的过程中发现无法下载`centos/7`的box，可以手动下载后将其添加到vagrant中。

**手动添加centos/7 box**

```bash
wget -c http://cloud.centos.org/centos/7/vagrant/x86_64/images/CentOS-7-x86_64-Vagrant-1801_02.VirtualBox.box
vagrant box add CentOS-7-x86_64-Vagrant-1801_02.VirtualBox.box --name centos/7
```

这样下次运行`vagrant up`的时候就会自动读取本地的`centos/7` box而不会再到网上下载。

### 访问kubernetes集群

访问Kubernetes集群的方式有三种：

- 本地访问
- 在VM内部访问
- Kubernetes dashboard

**通过本地访问**

可以直接在你自己的本地环境中操作该kubernetes集群，而无需登录到虚拟机中。

要想在本地直接操作Kubernetes集群，需要在你的电脑里安装`kubectl`命令行工具，对于Mac用户执行以下步骤：

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.11.0/kubernetes-client-darwin-amd64.tar.gz
tar xvf kubernetes-client-darwin-amd64.tar.gz && cp kubernetes/client/bin/kubectl /usr/local/bin
```

将`conf/admin.kubeconfig`文件放到`~/.kube/config`目录下即可在本地使用`kubectl`命令操作集群。

```bash
mkdir -p ~/.kube
cp conf/admin.kubeconfig ~/.kube/config
```

我们推荐您使用这种方式。

**在虚拟机内部访问**

如果有任何问题可以登录到虚拟机内部调试：

```bash
vagrant ssh node1
sudo -i
kubectl get nodes
```

**Kubernetes dashboard（可选安装）**

还可以直接通过dashboard UI来访问：[https://172.17.8.101:8443](https://172.17.8.101:8443/)

可以在本地执行以下命令获取token的值（需要提前安装kubectl）：

```bash
kubectl -n kube-system describe secret `kubectl -n kube-system get secret|grep admin-token|cut -d " " -f1`|grep "token:"|tr -s " "|cut -d " " -f2
```

**注意**：token的值也可以在`vagrant up`的日志的最后看到。

![Kubernetes dashboard](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster/raw/master/images/dashboard-animation.gif)

只有当你安装了下面的heapster组件后才能看到上图中的监控metrics。

### 组件

**Traefik**

部署Traefik ingress controller和增加ingress配置：

```bash
kubectl apply -f addon/traefik-ingress
```

在本地`/etc/hosts`中增加一条配置：

```bash
172.17.8.102 traefik.jimmysong.io
```

访问Traefik UI：[http://traefik.jimmysong.io](http://traefik.jimmysong.io/)

![Traefik Ingress controller](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster/raw/master/images/traefik-ingress.gif)

**Helm**

用来部署helm。

```bash
hack/deploy-helm.sh
```

### 安装 Istio

我们以安装 [istio](https://istio.io/) 1.0 为例。

**安装**

到 [Istio release](https://github.com/istio/istio/releases) 页面下载 istio 的安装包，安装 istio 命令行工具，将 `istioctl` 命令行工具放到你的`$PATH`目录下，对于 Mac 用户：

```bash
wget https://github.com/istio/istio/releases/download/1.0.0/istio-1.0.0-osx.tar.gz
tar xvf istio-1.0.0-osx.tar.gz
mv bin/istioctl /usr/local/bin/
```

在 Kubernetes 中部署 istio：

```bash
kubectl apply -f addon/istio/
```

**运行示例**

```bash
kubectl apply -n default -f <(istioctl kube-inject -f yaml/istio-bookinfo/bookinfo.yaml)
istioctl create -n default -f yaml/istio-bookinfo/bookinfo-gateway.yaml
```

在您自己的本地主机的`/etc/hosts`文件中增加如下配置项。

```bash
172.17.8.102 grafana.istio.jimmysong.io
172.17.8.102 servicegraph.istio.jimmysong.io
```

我们可以通过下面的URL地址访问以上的服务。

| Service      | URL                                                          |
| ------------ | ------------------------------------------------------------ |
| grafana      | [http://grafana.istio.jimmysong.io](http://grafana.istio.jimmysong.io/) |
| servicegraph | <http://servicegraph.istio.jimmysong.io/dotviz>, <http://servicegraph.istio.jimmysong.io/graph>,<http://servicegraph.istio.jimmysong.io/force/forcegraph.html> |
| tracing      | [http://172.17.8.101:31888](http://172.17.8.101:31888/)      |
| productpage  | <http://172.17.8.101:31380/productpage>                      |

详细信息请参阅 <https://istio.io/docs/guides/bookinfo.html>

![Bookinfo Demo](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster/raw/master/images/bookinfo-demo.gif)

### Vistio（可选安装）

[Vizceral](https://github.com/Netflix/vizceral)是Netflix发布的一个开源项目，用于近乎实时地监控应用程序和集群之间的网络流量。Vistio是使用Vizceral对Istio和网格监控的改进。它利用Istio Mixer生成的指标，然后将其输入Prometheus。Vistio查询Prometheus并将数据存储在本地以允许重播流量。

```bash
# Deploy vistio via kubectl
kubectl -n default apply -f addon/vistio/

# Expose vistio-api
kubectl -n default port-forward $(kubectl -n default get pod -l app=vistio-api -o jsonpath='{.items[0].metadata.name}') 9091:9091 &

# Expose vistio in another terminal window
kubectl -n default port-forward $(kubectl -n default get pod -l app=vistio-web -o jsonpath='{.items[0].metadata.name}') 8080:8080 &
```

如果一切都已经启动并准备就绪，您就可以访问Vistio UI，开始探索服务网格网络，访问[http://localhost:8080](http://localhost:8080/) 您将会看到类似下图的输出。

![vistio 页面](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster/raw/master/images/vistio-animation.gif)

更多详细内容请参考[Vistio—使用Netflix的Vizceral可视化Istio service mesh](http://www.servicemesher.com/blog/vistio-visualize-your-istio-mesh-using-netflixs-vizceral/)。

### Kiali（可选安装）

Kiali是一个用于提供Istio service mesh观察性的项目，更多信息请查看[https://kiali.io](https://kiali.io/)。

在本地该项目的根路径下执行下面的命令：

```bash
kubectl apply -n istio-system -f addon/kiali
```

![kiali](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster/raw/master/images/kiali.gif)

**注意**：当前还不支持jeager追踪，请使用上文中提到的jeager地址。

### Weave scope（可选安装）

[Weave scope](https://github.com/weaveworks/scope)可用于监控、可视化和管理Docker&Kubernetes集群，详情见<https://www.weave.works/oss/scope/>

在本地该项目的根路径下执行下面的命令：

```bash
kubectl apply -f addon/weave-scope
```

在本地的`/etc/hosts`下增加一条记录。

```bash
172.17.8.102 scope.weave.jimmysong.io
```

现在打开浏览器，访问<http://scope.weave.jimmysong.io/>

![Weave scope动画](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster/raw/master/images/weave-scope-animation.gif)

## 管理

除了特别说明，以下命令都在当前的repo目录下操作。

### 挂起

将当前的虚拟机挂起，以便下次恢复。

```bash
vagrant suspend
```

### 恢复

恢复虚拟机的上次状态。

```bash
vagrant resume
```

注意：我们每次挂起虚拟机后再重新启动它们的时候，看到的虚拟机中的时间依然是挂载时候的时间，这样将导致监控查看起来比较麻烦。因此请考虑先停机再重新启动虚拟机。

### 重启

停机后重启启动。

```bash
vagrant halt
vagrant up
# login to node1
vagrant ssh node1
# run the prosivision scripts
/vagrant/hack/k8s-init.sh
exit
# login to node2
vagrant ssh node2
# run the prosivision scripts
/vagrant/hack/k8s-init.sh
exit
# login to node3
vagrant ssh node3
# run the prosivision scripts
/vagrant/hack/k8s-init.sh
sudo -i
cd /vagrant/hack
./deploy-base-services.sh
exit
```

现在你已经拥有一个完整的基础的kubernetes运行环境，在该repo的根目录下执行下面的命令可以获取kubernetes dahsboard的admin用户的token。

```bash
hack/get-dashboard-token.sh
```

根据提示登录即可。

### 清理

清理虚拟机。

```bash
vagrant destroy
rm -rf .vagrant
```

## 参考

- [Kubernetes Handbook——Kubernetes中文指南/云原生应用架构实践手册](https://jimmysong.io/kubernetes-handbook - jimmysong.io)
- [rootsongjc/kubernetes-vagrant-centos-cluster - github.com](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster/blob/master/README-cn.md)
- [duffqiu/centos-vagrant - github.com](https://github.com/duffqiu/centos-vagrant)
- [coredns/deployment - github.com](https://github.com/coredns/deployment)
- [Kubernetes 1.8 kube-proxy 开启 ipvs - mritd.me](https://mritd.me/2017/10/10/kube-proxy-use-ipvs-on-kubernetes-1.8/#%E4%B8%80%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
- [Vistio—使用Netflix的Vizceral可视化Istio service mesh - servicemesher.com](http://www.servicemesher.com/blog/vistio-visualize-your-istio-mesh-using-netflixs-vizceral/)