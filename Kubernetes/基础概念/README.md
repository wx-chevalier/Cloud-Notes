# Kubernetes 基础概念

# Kubernetes 与 Docker

Docker 和 Kubernetes 的区别是什么？为什么说 Docker 要灭亡，而 Kubernetes 要兴起呢？Docker 指的是 Docker engine（也叫做 Docker daemon，或最新的名字：Moby），它是一种容器运行时（container runtime）的实现，而且是最主流的实现，几乎就是容器业界的事实标准。Docker 是用来创建和管理容器的，它和容器的关系就好比 Hypervisor（比如：KVM）和虚拟机之间的关系。当然，Docker 公司对 Docker engine 本身的定位和期望不仅仅在于在单机上管理容器，所以近年来一直在向 Docker engine 中加入各种各样的高级功能，比如：组建多节点的 Docker 集群、容器编排、服务发现，等等。而 Kubernetes，是搭建容器集群和进行容器编排的主流开源项目（由 Google 发起并维护），适合搭建 PaaS 平台。容器是 Kubernetes 管理的核心目标对象，它和容器的关系就好比 OpenStack 和虚拟机之间的关系，而它和 Docker 的关系就好比 OpenStack 和 Hypervisor 之间的关系。一般来说，Kubernetes 是和 Docker 配合使用的，Kubernetes 调用每个节点上的 Docker 去创建和管理容器，所以，你可以认为 Kubernetes 是大脑，而 Docker 是四肢。因此，在这样的背景下，Docker 逐渐式微，而 Kubernetes 崛起，就毫不奇怪了。

# 链接

- https://mp.weixin.qq.com/s/WC5TQSBHiHsAIDtpDsZ1qw
