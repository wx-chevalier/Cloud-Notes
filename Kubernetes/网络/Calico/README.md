# Calico

Calico 没有使用 CNI 的网桥模式，而是将节点当成边界路由器，组成了一个全连通的网络，通过 BGP 协议交换路由。所以，Calico 的 CNI 插件还需要在宿主机上为每个容器的 Veth Pair 设备配置一条路由规则，用于接收传入的 IP 包。Calico 的组件：

- CNI 插件：Calico 与 Kubernetes 对接的部分。
- Felix：负责在宿主机上插入路由规则（即：写入 Linux 内核的 FIB 转发信息库），以及维护 Calico 所需的网络设备等工作。
- BIRD (BGP Route Reflector)：是 BGP 的客户端，专门负责在集群里分发路由规则信息。

三个组件都是通过一个 DaemonSet 安装的。CNI 插件是通过 initContainer 安装的；而 Felix 和 BIRD 是同一个 pod 的两个 container。
