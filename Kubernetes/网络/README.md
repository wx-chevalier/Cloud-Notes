# Kubernetes 网络

在 Kubernetes 网络中存在两种 IP（Pod IP 和 Service Cluster IP），Pod IP 地址是实际存在于某个网卡(可以是虚拟设备)上的，Service Cluster IP 它是一个虚拟 IP，是由 kube-proxy 使用 Iptables 规则重新定向到其本地端口，再均衡到后端 Pod 的。每个 Pod 都拥有一个独立的 IP 地址（IPper Pod），而且假定所有的 pod 都在一个可以直接连通的、扁平的网络空间中。用户不需要额外考虑如何建立 Pod 之间的连接，也不需要考虑将容器端口映射到主机端口等问题。

同一个 Pod 的容器共享同一个网络命名空间，它们之间的访问可以用 localhost 地址 + 容器端口就可以访问。同一 Node 中 Pod 的默认路由都是 docker0 的地址，由于它们关联在同一个 docker0 网桥上，地址网段相同，所有它们之间应当是能直接通信的。不同 Node 中 Pod 间通信要满足 2 个条件：Pod 的 IP 不能冲突；将 Pod 的 IP 和所在的 Node 的 IP 关联起来，通过这个关联让 Pod 可以互相访问。

![image](https://user-images.githubusercontent.com/5803001/45594553-71001600-b9cf-11e8-83cf-d8755104e762.png)

# 网络解决方案

## 隧道方案（Overlay Networking）

隧道方案在 IaaS 层的网络中应用也比较多，大家共识是随着节点规模的增长复杂度会提升，而且出了网络问题跟踪起来比较麻烦，大规模集群情况下这是需要考虑的一个点。

- Weave：UDP 广播，本机建立新的 BR，通过 PCAP 互通

- Open vSwitch（OVS）：基于 VxLan 和 GRE 协议，但是性能方面损失比较严重

- Flannel：UDP 广播，VxLan

- Racher：IPsec

## 路由方案

路由方案一般是从 3 层或者 2 层实现隔离和跨主机容器互通的，出了问题也很容易排查。

- Calico：基于 BGP 协议的路由方案，支持很细致的 ACL 控制，对混合云亲和度比较高。

- Macvlan：从逻辑和 Kernel 层来看隔离性和性能最优的方案，基于二层隔离，所以需要二层路由器支持，大多数云服务商不支持，所以混合云上比较难以实现。
