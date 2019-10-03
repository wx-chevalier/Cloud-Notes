# Flannel

Flannel 是 CoreOS 团队针对 Kubernetes 设计的一个网络规划服务，简单来说，它的功能是让集群中的不同节点主机创建的 Docker 容器都具有全集群唯一的虚拟 IP 地址。Flannel 和 OpenVSwitch 思路基本一致，就是当 Docker 在宿主机上创建一个网桥的时候，用自己的网桥替代它。在默认的 Docker 配置中，每个节点上的 Docker 服务会分别负责所在节点容器的 IP 分配。这样导致的一个问题是，不同节点上容器可能获得相同的内外 IP 地址。并使这些容器之间能够之间通过 IP 地址相互找到，也就是相互 ping 通。

![Flannel 网络模型解析](https://i.postimg.cc/bNGshH3c/image.png)

Flannel 的设计目的就是为集群中的所有节点重新规划 IP 地址的使用规则，从而使得不同节点上的容器能够获得“同属一个内网”且”不重复的”IP 地址，并让属于不同节点上的容器能够直接通过内网 IP 通信。Flannel 实质上是一种“覆盖网络(overlaynetwork)”，也就是将 TCP 数据包装在另一种网络包里面进行路由转发和通信，目前已经支持 udp、vxlan、host-gw、aws-vpc、gce 和 alloc 路由等数据转发方式，默认的节点间数据通信方式是 UDP 转发。

![image](https://user-images.githubusercontent.com/5803001/45594693-8413e580-b9d1-11e8-90f0-cdf66129187d.png)

数据请求从容器 1(10.0.46.2:2379)中发出后，首先经由所在主机的 docker0 虚拟网卡(10.0.46.1)转发到 flannel0 虚拟网卡(10.0.46.0)，这是个 P2P 虚拟网卡，Flannel 通过修改 Node 路由表的方式实现 flanneld 服务监听 flannel0 虚拟网卡数据。

接着 flannel 服务将原本的数据内容 UDP 封装后根据自己的路由表投递给目的节点的 flanneld 服务。在此包中，包含有 outer-ip(source:192.168.8.227, dest:192.168.8.228)，inner-ip(source:10.0.46.2:2379, dest:10.0.90.2:8080)。

数据到达 node2 以后被解包，直接进入目的节点的 flannel0 虚拟网卡中(10.0.90.0)，且被转发到目的主机的 docker0 虚拟网卡(10.0.90.1)，最后就像本机容器通信一样由 docker0 路由到达目标容器 2(10.0.90.2:8080)。

为使每个结点上的容器分配的地址不冲突。Flannel 通过 Etcd 分配了每个节点可用的 IP 地址段后，再修改 Docker 的启动参数。“--bip=X.X.X.X/X”这个参数，它限制了所在节点容器获得的 IP 范围。
