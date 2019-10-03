# Pod

按照容器的设计理念，每个容器只运行单个进程。而要想实现多个 container 被绑定在一起进行管理的需求。我们需要一种高级别的概念来实现这个。在 kubernetes 中，这就是 Pod。在 Pod 里面，Container 之间可以共享网络（IP/Port）、共享存储（Volume）、共享 Hostname。

另外，Pod 可以理解成一个”逻辑主机”，它与非容器领域的物理主机或者 VM 有着类似的行为。在同一个 Pod 运行的进程就像在同一物理主机或 VM 上运行的进程一样。只是这些进程被单独的放到单个 container 内。

# 网络空间

在 Kubernetes 网络中存在两种 IP（Pod IP 和 Service Cluster IP），Pod IP 地址是实际存在于某个网卡(可以是虚拟设备)上的，Service Cluster IP 它是一个虚拟 IP，是由 kube-proxy 使用 Iptables 规则重新定向到其本地端口，再均衡到后端 Pod 的。每个 Pod 都拥有一个独立的 IP 地址（IPper Pod），而且假定所有的 pod 都在一个可以直接连通的、扁平的网络空间中。用户不需要额外考虑如何建立 Pod 之间的连接，也不需要考虑将容器端口映射到主机端口等问题。

同一个 Pod 的容器共享同一个网络命名空间，它们之间的访问可以用 localhost 地址 + 容器端口就可以访问。同一 Node 中 Pod 的默认路由都是 docker0 的地址，由于它们关联在同一个 docker0 网桥上，地址网段相同，所有它们之间应当是能直接通信的。不同 Node 中 Pod 间通信要满足 2 个条件： Pod 的 IP 不能冲突； 将 Pod 的 IP 和所在的 Node 的 IP 关联起来，通过这个关联让 Pod 可以互相访问。

![image](https://user-images.githubusercontent.com/5803001/45594553-71001600-b9cf-11e8-83cf-d8755104e762.png)

# 链接

- https://mp.weixin.qq.com/s/Vrgff_qfKWPFmRPb_LVMmQ
