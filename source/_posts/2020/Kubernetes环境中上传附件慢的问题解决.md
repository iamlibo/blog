---

title: Kubernetes环境中上传附件慢的问题解决
date: 2020-10-16 05:40:36
tags: [kubernetes,k8s]

---

最近项目中有一个上传文件的功能，文件大小基本上在100M以上，在使用过程中发现在有时候上传特别慢，内网环境100m需要几分钟，有时间却需要10秒。

<!-- more -->

环境情况：

- 环境是部署在公司内网Kubernetes的集群中
- 通过内网域名访问Kubernetes集群，前端使用Ingress进行反射代理
  
经过N轮的验证，最终得出来的结果：
内网域名地址与Pods所在的Node 是同一台机器，速度很快，如果不是一台机器速度就很慢。这也证明的不是程序的问题，应该从部署层面，重点是网络层面查找问题。

- 验证过程中排除了集群Node节点间的网络问题，节点间传输速度正常
- iptables 规则在可以接受范围内，不会导致性能下降太多

查看Kubernetes kube-proxy 组件的日志时发现一条Waring Can`t enable XDP acceleration. error=kernel is too old (have: 3.10.0 but want at least: 4.16.0)

只是一个警告，一开始没有在意，后来再没有其他信息，只能升级内核版本试一下。将Master 与Node 节点的内核都升级到5.8版本。启动集群没有Waring 信息出现。验证上传功能，正常！

参考资料：
[xdp](https://www.iovisor.org/technology/xdp)