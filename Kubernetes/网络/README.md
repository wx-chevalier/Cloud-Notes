# Kubernetes 网络

容器网络发展到现在，形成了两大阵营，就是 Docker 的 CNM 和 Google、CoreOS、Kuberenetes 主导的 CNI。首先明确一点，CNM 和 CNI 并不是网络实现，他们是网络规范和网络体系，从研发的角度他们就是一堆接口，你底层是用 Flannel 也好、用 Calico 也好，他们并不关心，CNM 和 CNI 关心的是网络管理的问题。

- CNM（Container Network Model）: Docker Libnetwork 的优势就是原生，而且和 Docker 容器生命周期结合紧密；缺点也可以理解为是原生，被 Docker 绑架。支持 CNM 的网络模型包括了 Docker Swarm overlay, Macvlan & IP networkdrivers, Calico, Contiv, Weave.

- CNI 的优势是兼容其他容器技术（e.g. rkt）及上层编排系统（Kubernetes & Mesos)，而且社区活跃势头迅猛，Kubernetes 加上 CoreOS 主推；缺点是非 Docker 原生。支持 CNI 的网络模型包括了 Kubernetes, Weave, Macvlan, Calico, Flannel, Contiv, Mesos CNI.

# 网络解决方案

Kubernetes 的集群网络插件实现方案有很多种，社区比较常见的有两种 Flannel 和 Calico 的工作原理。

## 隧道方案（Overlay Networking）

隧道方案在 IaaS 层的网络中应用也比较多，大家共识是随着节点规模的增长复杂度会提升，而且出了网络问题跟踪起来比较麻烦，大规模集群情况下这是需要考虑的一个点。

- Weave：UDP 广播，本机建立新的 BR，通过 PCAP 互通
- Open vSwitch（OVS）：基于 VxLan 和 GRE 协议，但是性能方面损失比较严重
- Flannel：Flannel 主要提供了 Overlay 的网络方案，UDP 模式由于其封包拆包的过程涉及了多次上下文的切换，导致性能很差，逐渐被社区抛弃；VXLAN 模式的封包拆包过程均在内核态，性能要比 UDP 好很多，也是最经常使用的模式；host-gw 模式不涉及封包拆包，所以性能相对较高，但要求节点间二层互通。
- Racher：IPsec

## 路由方案

路由方案一般是从 3 层或者 2 层实现隔离和跨主机容器互通的，出了问题也很容易排查。

- Calico：Calico 主要采用了 BGP 协议交换路由，没有采用 cni0 网桥，当二层网络不通的时候，可以采用 IPIP 模式，但由于涉及到封包拆包的过程，性能相对较弱，与 Flannel 的 VXLAN 模式相当。Calico 支持很细致的 ACL 控制，对混合云亲和度比较高。
- Macvlan：从逻辑和 Kernel 层来看隔离性和性能最优的方案，基于二层隔离，所以需要二层路由器支持，大多数云服务商不支持，所以混合云上比较难以实现。
