# Service

RC、RS 和 Deployment 只是保证了支撑服务的微服务 Pod 的数量，但是没有解决如何访问这些服务的问题。如果说 Deployment 是负责保证 Pod 组的正常运行，那么 Service 就是用于保证以合理的网络来连接到该组 Pod。一个 Pod 只是一个运行服务的实例，随时可能在一个节点上停止，在另一个节点以一个新的 IP 启动一个新的 Pod，因此不能以确定的 IP 和端口号提供服务。要稳定地提供服务需要服务发现和负载均衡能力。服务发现完成的工作，是针对客户端访问的服务，找到对应的的后端服务实例。

Service 是对一组提供相同功能的 Pods 的抽象，并为它们提供一个统一的入口。借助 Service，应用可以方便的实现服务发现与负载均衡，并实现应用的零宕机升级。譬如考虑一个图片处理后端应用程序，它运行了 3 个副本。这些副本是可互换的：frontend 不需要关心它们调用了哪个 backend 副本。然而组成这一组 backend 程序的 Pod 实际上可能会发生变化，frontend 客户端不应该也没必要知道，而且也不需要跟踪这一组 backend 的状态；Service 定义的抽象能够解耦这种关联。

# 域名解析

Kubernetes 将 Service Name 与 Service Cluster IP 做一个 DNS 域名映射，优雅的解决了服务发现的问题。Kubernetes 提供了内置的 dns 机制和 ClusterIP 机制，每个 Service 都自动注册域名，分配 ClusterIP，这样服务间的依赖可以从 IP 变为 name。DNS server 通过 kubernetes api server 来观测是否有新 Service 建立，并为其建立对应的 dns 记录。如果集群已经 enable DNS，那么 Pod 可以自动对 Service 做 name 解析。

例：有个叫做”my-service“的 service，他对应的 kubernetes namespace 为”my-ns“，那么会有他对应的 dns 记录，叫做”my-service.my-ns“。那么在 my-ns 的 namespace 中的 pod 都可以对 my-service 做 name 解析来轻松找到这个 service。在其他 namespace 中的 pod 解析”my-service.my-ns“来找到他。解析出来的结果是这个 service 对应的 cluster ip。

# 应用选择

Kubernetes Service 能够支持 TCP 和 UDP 协议，默认 TCP 协议，其通过标签来选取服务后端，一般配合 Replication Controller 或者 Deployment 来保证后端容器的正常运行。这一组 Pod 能够被 Service 访问到，通常是通过 Label Selector 实现的。另外，也可以将已有的服务以 Service 的形式加入到 Kubernetes 集群中来，只需要在创建 Service 的时候不指定 Label selector，而是在 Service 创建好后手动为其添加 endpoint。对 Kubernetes 集群中的应用，Kubernetes 提供了简单的 Endpoints API，只要 Service 中的一组 Pod 发生变更，应用程序就会被更新。对非 Kubernetes 集群中的应用，Kubernetes 提供了基于 VIP 的网桥的方式访问 Service，再由 Service 重定向到 backend Pod。

# 负载均衡

在 Kubernetes 集群中微服务的负载均衡是由 kube-proxy 实现的。kube-proxy 是 Kubernetes 集群内部的负载均衡器。它是一个分布式代理服务器，在 Kubernetes 的每个节点上都有一个；这一设计体现了它的伸缩性优势，需要访问服务的节点越多，提供负载均衡能力的 Kube-proxy 就越多，高可用节点也随之增多。与之相比，我们平时在服务器端做个反向代理做负载均衡，还要进一步解决反向代理的负载均衡和高可用问题。

kubernetes 中通过 service 概念来对应用做多 POD 间的负载均衡，Service 是一组 Pod 的服务抽象，相当于一组 Pod 的 LB，负责将请求分发给对应的 Pod；Service 会为这个 LB 提供一个 IP，一般称为 ClusterIP。Service Cluster IP 是 Kubernetes 系统中的虚拟 IP 地址，由系统动态分配；Kubernetes 集群中的每个节点都会运行 kube-proxy，其负责为 ExternalName 以外的服务实现虚拟 IP 形式，v1.2 版本后默认使用的是 iptables。

Kube-proxy 是一个简单的网络代理和负载均衡器，它的作用主要是负责 Service 的实现，具体来说，就是实现了内部从 Pod 到 Service 和外部的从 NodePort 向 Service 的访问。

![image](https://user-images.githubusercontent.com/5803001/45594895-ff2acb00-b9d4-11e8-89ed-a7b0f724c249.png)

kubernetes 中的每个 node 都会运行一个 kube-proxy。他为每个 service 都映射一个本地 port，任何连接这个本地 port 的请求都会转到 backend 后的随机一个 pod，service 中的字段 SessionAffinity 决定了使用 backend 的哪个 pod，最后在本地建立一些 iptables 规则，这样访问 service 的 cluster ip 以及对应的 port 时，就能将请求映射到后端的 pod 中。

![image](https://user-images.githubusercontent.com/5803001/45594757-832f8380-b9d2-11e8-9e61-b696f63051fd.png)

在这种模式下，kube-proxy 监视 Kubernetes 主服务器添加和删除服务和端点对象。对于每个服务，它安装 iptables 规则，捕获到服务的 clusterIP（虚拟）和端口的流量，并将流量重定向到服务的后端集合之一。对于每个 Endpoints 对象，它安装选择后端 Pod 的 iptables 规则。默认情况下，后端的选择是随机的。可以通过将 service.spec.sessionAffinity 设置为“ClientIP”（默认为“无”）来选择基于客户端 IP 的会话关联。与用户空间代理一样，最终结果是绑定到服务的 IP:端口的任何流量被代理到适当的后端，而客户端不知道关于 Kubernetes 或服务或 Pod 的任何信息。这应该比用户空间代理更快，更可靠。然而，与用户空间代理不同，如果最初选择的 Pod 不响应，则 iptables 代理不能自动重试另一个 Pod，因此它取决于具有工作准备就绪探测。

# 链接

- https://parg.co/kXe
