# 服务

Kubernetes Pod 是有生命周期的，它们可以被创建，也可以被销毁，然后一旦被销毁生命就永远结束。通过 ReplicationController 能够动态地创建和销毁 Pod（列如，需要进行扩缩容，或者执行滚动升级）；每个 Pod 都会获取它自己的 IP 地址，即使这些 IP 地址不总是稳定可依赖的。这会导致一个问题；在 Kubernetes 集群中，如果一组 Pod（称为 backend）为其他 Pod（称为 frontend）提供服务，那么哪些 frontend 该如何发现，并连接到这组 Pod 中的那些 backend 就成了极大的挑战。

> A Kubernetes Service is an abstraction layer which defines a logical set of Pods and enables external traffic exposure, load balancing and service discovery for those Pods.

Kubernetes Service 定义了这样一种抽象：一个 Pod 的逻辑分组及一种可以访问它们不同的策略；即 Service 是对一组提供相同功能的 Pods 的抽象，并为它们提供一个统一的入口。借助 Service，应用可以方便的实现服务发现与负载均衡，并实现应用的零宕机升级。譬如考虑一个图片处理后端应用程序，它运行了 3 个副本。这些副本是可互换的：frontend 不需要关心它们调用了哪个 backend 副本。然而组成这一组 backend 程序的 Pod 实际上可能会发生变化，frontend 客户端不应该也没必要知道，而且也不需要跟踪这一组 backend 的状态；Service 定义的抽象能够解耦这种关联。

在 K8s 集群中，客户端需要访问的服务就是 Service 对象。每个 Service 会对应一个集群内部有效的虚拟 IP，集群内部通过虚拟 IP 访问一个服务。Service 由 kube-proxy 实现软件负载均衡器，负责将对 Service 的请求转发到后端的某个 Pod 实例上。且 Kubernetes 为每个 Service 分配了一个全局唯一的虚拟 IP 地址(Cluster IP)，每个服务在 Kubernetes 架构上即变成了具备唯一 IP 地址的通信节点。此外，K8s 还内建了基于域名的服务发现机制，Kubernetes 将 Service Name 与 Service Cluster IP 做一个 DNS 域名映射，优雅的解决了服务发现的问题。Kubernetes 提供了内置的 dns 机制和 ClusterIP 机制，每个 Service 都自动注册域名，分配 Cluster IP，这样服务间的依赖可以从 IP 变为 name。DNS Server 通过 K8s api server 来观测是否有新 Service 建立，并为其建立对应的 dns 记录。如果集群已经 enable DNS，那么 Pod 可以自动对 Service 做 name 解析。

譬如，有个叫做 my-service 的 Service，他对应的 kubernetes namespace 为 my-ns，那么会有他对应的 dns 记录，叫做 my-service.my-ns。那么在 my-ns 的 namespace 中的 Pod 都可以对 my-service 做 name 解析来轻松找到这个 Service。在其他 namespace 中的 pod 解析 my-service.my-ns 来找到他。解析出来的结果是这个 Service 对应的 Cluster IP。

# 链接

- https://parg.co/kXe
