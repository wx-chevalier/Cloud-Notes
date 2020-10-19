# UDP 模式

UDP 模式，是 Flannel 项目最早支持的一种方式，却也是性能最差的一种方式。这种模式提供的是一个三层的 Overlay 网络，即：它首先对发出端的 IP 包进行 UDP 封装，然后在接收端进行解封装拿到原始的 IP 包，进而把这个 IP 包转发给目标容器。工作原理如下图所示。

![Flannel UDP 模式示意](https://s1.ax1x.com/2020/10/19/0vHaKU.png)

Node 1 上的 Pod 1 请求 Node2 上的 Pod 2 时，流量的走向如下：

1. Pod 1 里的进程发起请求，发出 IP 包；
2. IP 包根据 Pod 1 里的 veth 设备对，进入到 cni0 网桥；
3. 由于 IP 包的目的 ip 不在 Node 1 上，根据 flannel 在节点上创建出来的路由规则，进入到 flannel0 中；
4. 此时 flanneld 进程会收到这个包，flanneld 判断该包应该在哪台 node 上，然后将其封装在一个 UDP 包中；
5. 最后通过 Node 1 上的网关，发送给 Node2；

flannel0 是一个 TUN 设备（Tunnel 设备）。在 Linux 中，TUN 设备是一种工作在三层（Network Layer）的虚拟网络设备。TUN 设备的功能：在操作系统内核和用户应用程序之间传递 IP 包。

可以看到，这种模式性能差的原因在于，整个包的 UDP 封装过程是 flanneld 程序做的，也就是用户态，而这就带来了一次内核态向用户态的转换，以及一次用户态向内核态的转换。在上下文切换和用户态操作的代价其实是比较高的，而 UDP 模式因为封包拆包带来了额外的性能消耗。
