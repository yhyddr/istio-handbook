# 流量管理

这一章节将大家了解 Istio 流量管理中的各种概念的含义及表示方法。

流量管理是 Isito 中的最基础功能，使用 Istio 的流量管理模型，本质上是将流量与基础设施扩容解耦，让运维人员可以通过 Pilot 指定流量遵循什么规则，而不是指定哪些 pod/VM 应该接收流量——Pilot 和智能 Envoy 代理会帮我们搞定。关于流量管理的详细介绍请参考 [Istio 官方文档](https://istio.io/zh/docs/concepts/traffic-management/)。

## 参考

- [流量管理 - istio.io](https://istio.io/zh/docs/concepts/traffic-management/)