# Istio 安装

Istio 官方推荐使用 Helm 来安装，Istio 中的很多组件都可以选择安装或开启，因此 Helm chart 也是组合式的，[下载 Istio 安装包](https://github.com/istio/istio/releases)后解压可以看到 `install/kubernetes/helm/istio` 目录下的 Helm chart 配置文件，参考 [使用 Helm 进行安装](https://istio.io/zh/docs/setup/kubernetes/helm-install/)。

## 使用 Helm 安装

下图是 Istio Helm Chart 的配置大全，通过给不同的组件使用不同的着色给大家一个直观的体验。

![Istio Helm Chart Cheatsheet(原图来自沈旭光)](../images/istio-chart-cheatsheet.jpg)

上图可以通过 [Google doc](https://docs.google.com/spreadsheets/d/14eXerRWNsCJDUrKWjoQIwwxbB8TnvrbgQQvC0RmWFD4/edit#gid=0) 下载电子表格。

Istio 的安装文件中包括如下几个子 chart。

- ingress
- ingressgateway
- egressgateway
- sidecarInjectorWebhook
- galley
- mixer
- pilot
- security(citadel)
- grafana
- prometheus
- servicegraph
- tracing(jaeger)
- kiali

所有的这些子 Chart 都可以通过 YAML 配置中的 `enabled` 标志选择性的开启，具体配置方法请参考安装包解压后的 `install/kubernetes/helm/istio/README.md` 文件。

## Istio 中的 CRD

安装完 Istio 后我们再查看下 Istio 创建的与网络相关的 Kubernetes CRD（自定义资源类型），请参考使用自定义资源扩展 API。Istio 创建的所有的 Kubernetes CRD 可以这样查看。

```bash
kubectl get customresourcedefinition|grep istio.io
```

你将会看到 50 个 CRD，其实要想了解 Istio 控制平面是怎样工作的，只需要了解这 50 个 CRD 是怎么工作的即可，很遗憾目前 Istio 还没有推出 API 文档。根据 API 的域名看到所有的 CRD 分为四类。

- authentication：策略管控
- config：配置分发与遥测
- networking：流量管理
- rbac：基于角色的访问控制

详细列表如下：

```ini
# authentication，这两个 CRD 不是直接在 YAML 里定义的
meshpolicies.authentication.istio.io
policies.authentication.istio.io

# config
adapters.config.istio.io
apikeys.config.istio.io
attributemanifests.config.istio.io
authorizations.config.istio.io
bypasses.config.istio.io
checknothings.config.istio.io
circonuses.config.istio.io
deniers.config.istio.io
edges.config.istio.io
fluentds.config.istio.io
handlers.config.istio.io
httpapispecbindings.config.istio.io
httpapispecs.config.istio.io
instances.config.istio.io
kubernetesenvs.config.istio.io
kuberneteses.config.istio.io
listcheckers.config.istio.io
listentries.config.istio.io
logentries.config.istio.io
memquotas.config.istio.io
metrics.config.istio.io
noops.config.istio.io
opas.config.istio.io
prometheuses.config.istio.io
quotas.config.istio.io
quotaspecbindings.config.istio.io
quotaspecs.config.istio.io
rbacs.config.istio.io
redisquotas.config.istio.io
reportnothings.config.istio.io
rules.config.istio.io
servicecontrolreports.config.istio.io
servicecontrols.config.istio.io
signalfxs.config.istio.io
solarwindses.config.istio.io
stackdrivers.config.istio.io
statsds.config.istio.io
stdios.config.istio.io
templates.config.istio.io
tracespans.config.istio.io

# networking
destinationrules.networking.istio.io
envoyfilters.networking.istio.io
gateways.networking.istio.io
serviceentries.networking.istio.io
virtualservices.networking.istio.io

# rbac
rbacconfigs.rbac.istio.io
servicerolebindings.rbac.istio.io
serviceroles.rbac.istio.io
```

从中可以看出 `config` 类型的 CRD 是最多的，这是因为在 Mixer 中有众多的 adapter 导致，几十个 adapter 分别创建自己的适配器来对接基础设施后端。

CRD 的详细分类和用途如下图所示。

![Istio CRD Cheatsheet(原图来自沈旭光)](../images/istio-crd-cheatsheet.png)

上图可以通过 [Google doc](https://docs.google.com/spreadsheets/d/14eXerRWNsCJDUrKWjoQIwwxbB8TnvrbgQQvC0RmWFD4/edit#gid=0) 下载电子表格。

## 参考

- [使用 Helm 进行安装 - istio.io](https://istio.io/zh/docs/setup/kubernetes/helm-install/)