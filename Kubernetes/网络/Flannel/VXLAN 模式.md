# VXLAN 模式

VXLAN，即 Virtual Extensible LAN（虚拟可扩展局域网），是 Linux 内核本身就支持的一种网络虚拟化技术。通过利用 Linux 内核的这种特性，也可以实现在内核态的封装和解封装的能力，从而构建出覆盖网络。其工作原理如下图所示：

![VXLAN 工作模式](https://s1.ax1x.com/2020/10/19/0vb7TJ.png)

VXLAN 模式的 flannel 会在节点上创建一个叫 flannel.1 的 VTEP (VXLAN Tunnel End Point，虚拟隧道端点) 设备，跟 UDP 模式一样，该设备将二层数据帧封装在 UDP 包里，再转发出去，而与 UDP 模式不一样的是，整个封装的过程是在内核态完成的。

Node 1 上的 Pod 1 请求 Node 2 上的 Pod 2 时，流量的走向如下：

1. Pod 1 里的进程发起请求，发出 IP 包；
2. IP 包根据 Pod 1 里的 veth 设备对，进入到 cni0 网桥；
3. 由于 IP 包的目的 ip 不在 Node 1 上，根据 flannel 在节点上创建出来的路由规则，进入到 flannel.1 中；
4. flannel.1 将原始 IP 包加上一个目的 MAC 地址，封装成一个二层数据帧；然后内核将数据帧封装进一个 UDP 包里；
5. 最后通过 Node 1 上的网关，发送给 Node 2；
