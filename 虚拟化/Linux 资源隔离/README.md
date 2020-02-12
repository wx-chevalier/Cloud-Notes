# 容器技术与资源隔离

Linux cgroups 为一种名为 Linux 容器（LXC）的技术铺平了道路。LXC 实际上是我们今天所知的第一个实现容器的主要实现，利用 cgroup 和命名空间隔离来创建具有独立进程和网络空间的虚拟环境。从某种意义上说，这允许独立和隔离的用户空间。  容器的概念直接来自 LXC。事实上，早期版本的 Docker 直接构建在 LXC 之上。

简单的讲就是，Linux namespace 允许用户在独立进程之间隔离 CPU 等资源。进程的访问权限及可见性仅限于其所在的 Namespaces。因此，用户无需担心在一个 Namespace 内运行的进程与在另一个 Namespace 内运行的进程冲突。甚至可以同一台机器上的不同容器中运行具有相同 PID 的进程。同样的，两个不同容器中的应用程序可以使用相同的端口。

![](https://tva1.sinaimg.cn/large/007rAy9hgy1g2zdhwngx6j30u00m0wgg.jpg)

与虚拟机相比，容器更轻量且速度更快，因为它利用了 Linux 底层操作系统在隔离的环境中运行。虚拟机的 hypervisor 创建了一个非常牢固的边界，以防止应用程序突破它，而容器的边界不那么强大。另一个区别是，由于 Namespace 和 Cgroups 功能仅在 Linux 上可用，因此容器无法在其他操作系统上运行；Docker 实际上使用了一个技巧，并在非 Linux 操作系统上安装 Linux 虚拟机，然后在虚拟机内运行容器。

虽然大多数 IT 行业正在采用基于容器的基础架构（云原生解决方案），但必须了解该技术的局限性。传统容器（如 Docker，Linux Containers（LXC）和 Rocket（rkt））并不是真正的沙箱，因为它们共享主机操作系统内核。它们具有资源效率，但攻击面和破坏的潜在影响仍然很大，特别是在多租户云环境中，共同定位属于不同客户的容器。

当主机操作系统为每个容器创建虚拟化用户空间时，问题的根源是容器之间的弱分离。一直致力于设计真正的沙盒容器的研究和开发。大多数解决方案重新构建容器之间的边界以加强隔离。譬如来自 IBM，Google，Amazon 和 OpenStack 的四个独特项目，这些项目使用不同的技术来实现相同的目标，为容器创建更强的隔离。

- IBM Nabla 在 Unikernels 之上构建容器
- Google gVisor 创建了一个用于运行容器的专用客户机内核
- Amazon Firecracker 是一个用于沙盒应用程序的极轻量级管理程序
- OpenStack 将容器放置在针对容器编排平台优化的专用 VM 中

LXC
一种操作系统级虚拟化方法，在执行时不用重复加载内核, 且其内核与宿主共享，允许其他一些沙盒进程运行在一块相对独立的空间，并且能够方便的控制他们的资源调度。

namespace
通过 namespace 实现容器间的隔离性。容器内的应用只能在自己的命名空间中运行而且不会访问到命名空间之外。

cgroups（Control Groups)
用来管理群组。使应用隔离运行的关键是让它们只使用你想要的资源。这样可以确保在机器上运行的容器都是良民(good multi-tenant citizens)。群组控制允许 Docker 分享或者限制容器使用硬件资源。

AUFS (AnotherUnionFS)
一种 Union FS, 支持将不同目录挂载到同一个虚拟文件系统下的文件系统。

容器运行时的只读模板。每一个镜像由一系列的层 (layers) 组成，层是由 Dockerfile 指定。copy on write
写时复制。容器是由镜像所创建，会根据多层文件系统构建一个镜像栈，只有栈的最顶层是读写层。如果发生对只读层的写操作时会将该文件复制到读写层，并隐藏只读层的文件。

联合文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into asingle virtual filesystem)。
联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。
另外，不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。
Docker 中使用的 AUFS（AnotherUnionFS）就是一种联合文件系统。AUFS 支持为每一个成员目录（类似 Git 的分支）设定只读（readonly）、读写（readwrite）和写出（whiteout-able）权限, 同时 AUFS 里有一个类似分层的概念, 对只读权限的分支可以逻辑上进行增量地修改(不影响只读部分的)。
Docker 目前支持的联合文件系统种类包括 AUFS, btrfs, vfs 和 DeviceMapper

要想实现网络通信，机器至少需要一个网络接口(物理接口或虚拟接口)与外界相通，并可以收发数据包；另外，如果不同子网之间要进行通信，则需要额外的路由机制。Docker 的网络接口默认都是虚拟接口。虚拟接口的最大优势就是转发效率极高！之所以会这样，那是因为 Linux 通过在内核中进行数据复制来实现虚拟接口间的数据转发，即直接复制发送接口的发送缓存中的数据包到接收接口的接收缓存中，而无需通过外部物理网络设备进行交换。对于本地系统和容器内系统来看，虚拟接口和一个正常的以太网卡相比并无区别，只是虚拟接口的速度要快得多。

2.5.2 网络创建过程
创建一对虚拟接口，分别放到宿主机和容器的命名空间中；
宿主机一端的虚拟接口连接到默认的 docker0 网桥或指定网桥上，并具有一个以 veth 开头的唯一的名字;
容器一端的虚拟接口将被放到容器中，并修改名称为 eth0，且这个接口只对该容器的命名空间可见；4. 从网桥可用地址段中获取一个空闲的地址分配给容器的 eth0(如 172.17.0.2/16)，并配置默认路由网关为 docker0 网卡的内部接口 docker0 的 IP 地址(如 172.17.42.1/16)；
完成以上这些，容器就可以使用自身可见的 eth0 虚拟网卡来连接其他容器和访问外部网络。另外，可以在容器创建启动时通过--net 参数来指定容器的网络配置

# Cgroup

# Network

Docker 通过 libnetwork 实现了 CNM 网络模型。libnetwork 设计 doc 中对 CNM 模型的简单诠释如下：

![image](https://user-images.githubusercontent.com/5803001/45594781-e6211a80-b9d2-11e8-8252-3d4f52277a17.png)

CNM 模型有三个组件：

Sandbox(沙盒)：每个沙盒包含一个容器网络栈(network stack)的配置，配置包括：容器的网口、路由表和 DNS 设置等。
Endpoint(端点)：通过 Endpoint，沙盒可以被加入到一个 Network 里。
Network(网络)：一组能相互直接通信的 Endpoints。

CNM 模型在 Linux 上的参考实现技术，比如：沙盒的实现可以是一个 Linux Network Namespace；Endpoint 可以是一对 VETH；Network 则可以用 Linux Bridge 或 Vxlan 实

veth 对只是不同网络命名空间通信的一种解决方案，还有其他方案。

Linux Bridge，即 Linux 网桥设备，是 Linux 提供的一种虚拟网络设备之一。其工作方式非常类似于物理的网络交换机设备。Linux Bridge 可以工作在二

层，也可以工作在三层，默认工作在二层。工作在二层时，可以在同一网络的不同主机间转发以太网报文；一旦你给一个 Linux Bridge 分配了 IP 地址，

也就开启了该 Bridge 的三层工作模式。在 Linux 下，你可以用 iproute2 工具包或 brctl 命令对 Linux bridge 进行管理。

VETH(Virtual Ethernet )是 Linux 提供的另外一种特殊的网络设备，中文称为虚拟网卡接口。它总是成对出现，要创建就创建一个 pair。一个 Pair 中的

veth 就像一个网络线缆的两个端点，数据从一个端点进入，必然从另外一个端点流出。每个 veth 都可以被赋予 IP 地址，并参与三层网络路由过程。Network namespace，网络名字空间，允许你在 Linux 创建相互隔离的网络视图，每个网络名字空间都有独立的网络配置，比如：网络设备、路由表

等。新建的网络名字空间与主机默认网络名字空间之间是隔离的。我们平时默认操作的是主机的默认网络名字空间。

![image](https://user-images.githubusercontent.com/5803001/45594763-b5d97c00-b9d2-11e8-9001-377d8957d488.png)

# Todos

- [ ] https://jiajially.gitbooks.io/dockerguide/content/dockerCoreNS.html 提取其中的命令与原理解析

# 容器技术

**Machine-level virtualization**, such as [KVM](https://www.linux-kvm.org/) and [Xen](https://www.xenproject.org/), exposes virtualized hardware to a guest kernel via a Virtual Machine Monitor (VMM). This virtualized hardware is generally enlightened (paravirtualized) and additional mechanisms can be used to improve the visibility between the guest and host (e.g. balloon drivers, paravirtualized spinlocks). Running containers in distinct virtual machines can provide great isolation, compatibility and performance (though nested virtualization may bring challenges in this area), but for containers it often requires additional proxies and agents, and may require a larger resource footprint and slower start-up times.

[![Machine-level virtualization](https://github.com/google/gvisor/raw/master/g3doc/Machine-Virtualization.png)](https://github.com/google/gvisor/blob/master/g3doc/Machine-Virtualization.png)

**Rule-based execution**, such as [seccomp](https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt), [SELinux](https://selinuxproject.org/) and [AppArmor](https://wiki.ubuntu.com/AppArmor), allows the specification of a fine-grained security policy for an application or container. These schemes typically rely on hooks implemented inside the host kernel to enforce the rules. If the surface can be made small enough (i.e. a sufficiently complete policy defined), then this is an excellent way to sandbox applications and maintain native performance. However, in practice it can be extremely difficult (if not impossible) to reliably define a policy for arbitrary, previously unknown applications, making this approach challenging to apply universally.

[![Rule-based execution](https://github.com/google/gvisor/raw/master/g3doc/Rule-Based-Execution.png)](https://github.com/google/gvisor/blob/master/g3doc/Rule-Based-Execution.png)

Rule-based execution is often combined with additional layers for defense-in-depth.

**gVisor** provides a third isolation mechanism, distinct from those mentioned above.

gVisor intercepts application system calls and acts as the guest kernel, without the need for translation through virtualized hardware. gVisor may be thought of as either a merged guest kernel and VMM, or as seccomp on steroids. This architecture allows it to provide a flexible resource footprint (i.e. one based on threads and memory mappings, not fixed guest physical resources) while also lowering the fixed costs of virtualization. However, this comes at the price of reduced application compatibility and higher per-system call overhead.

[![gVisor](https://github.com/google/gvisor/raw/master/g3doc/Layers.png)](https://github.com/google/gvisor/blob/master/g3doc/Layers.png)

On top of this, gVisor employs rule-based execution to provide defense-in-depth (details below).

gVisor's approach is similar to [User Mode Linux (UML)](http://user-mode-linux.sourceforge.net/), although UML virtualizes hardware internally and thus provides a fixed resource footprint.

# 链接

- https://unit42.paloaltonetworks.com/making-containers-more-isolated-an-overview-of-sandboxed-container-technologies
