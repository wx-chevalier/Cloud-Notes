![Kubernetes 网络示意图](https://i.postimg.cc/cC1YKzNS/image.png)

# K8s 部署与实战

K8s 的官方定义为：K8s is an open-source system for automating deployment, scaling, and management of containerized applications.It groups containers that make up an application into logical units for easy management and discovery.

![K8s 概念图](https://i.postimg.cc/BQSNR1yd/image.png)

K8s [koo-ber-nay'-tice] 是 Google 基于 Borg 开源的容器编排调度引擎，其支持多种底层容器虚拟化技术，具有完备的功能用于支撑分布式系统以及微服务架构，同时具备超强的横向扩容能力；它提供了自动化容器的部署和复制，随时扩展或收缩容器规模，将容器组织成组，并且提供容器间的负载均衡，提供容器弹性等特性。作为 CNCF（Cloud Native Computing Foundation）最重要的组件之一，可谓云操作系统；它的目标不仅仅是一个编排系统，而是提供一个规范，可以让你来描述集群的架构，定义服务的最终状态。

## 背景

几乎所有在谷歌开发的软件都是在容器中运行的。谷歌大规模管理容器已经有十多年的时间了，当时人们对此谈论较少。为了满足内部需求，谷歌的一些开发人员已经建立了三个不同的容器管理系统。Borg , Omega 和 Kubernetes 。每一个系统的开发都受到了前者的极大影响，尽管它的开发原因不同。

谷歌开发的第一个容器管理系统是 Borg，它是为了管理长期服务和批处理作业而建立的，而这些工作之前是由两个系统处理的。Babysitter 和 Global Work Queue 。后者强烈影响了 Borg 的架构，但专注于运行批处理作业。Borg 仍然是谷歌内部最顶级的容器管理系统，因为它的规模、功能的多样性和极强的健壮性。第二个系统是 Omega，是 Borg 的后代。它是由改善 Borg 生态系统中的软件工程的愿望所驱动的。这个系统应用了许多在博格成功的标准，但它是从基础上建立起来的，以拥有最一致的架构。Omega 的许多创新后来都被纳入了 Borg。

第三个系统是 Kubernetes。在外部开发者对容器感兴趣的情况下孕育和发展起来的，谷歌今天发展了一项快速增长的业务，那就是销售公共云基础设施。Kubernetes 是开源的--与 Borg 和 Omega 相反，它们是作为谷歌内部的纯系统开发的。Kubernetes 的开发更注重那些编写在集群中运行的应用程序的开发人员的经验：它的主要目标是促进分布式系统的部署和管理，同时受益于容器使其能够最好地利用内存和处理资源。

# K8s 架构概览

和其他可用的编排器一样，k8s 也遵循主/从模式，从而构成一个集群，在这个集群中，它的运行必须至少有三个节点：主节点，负责（默认情况下）管理集群，其他节点作为工作者，是我们要在这个集群上运行的应用程序的执行者。K8s 的典型架构如下所示：

![K8s 架构](https://s3.ax1x.com/2021/02/14/yyZeoV.png)

- API Server：它是 k8s 的主要组成部分之一。该组件提供了一个通过 HTTP 使用 JSON 进行通信的 API，其中主要是由管理员使用 kubectl 工具与其他节点进行通信，如图所示。这些组件之间的通信是通过 REST 请求建立的。
- etcd：etcd 是一个分布式的键值数据存储，k8s 用来存储集群规格、状态和配置。所有存储在 etcd 中的数据都只能通过 API 来操作。出于安全考虑，etcd 默认只在 k8s 集群中被列为主控的节点上运行，但它们也可以在外部的、etcd 专用的集群上运行，例如。
- Scheduler：调度器负责选择承载特定 pod（k8s 集群中最小的单元--暂时不用担心这个问题，我们以后再谈）的节点来执行。这种选择是根据每个节点的可用资源量，以及集群中每个节点的状态进行选择，从而保证资源的良好分布。此外，选择节点，在其中执行一个或多个豆荚，还可以考虑用户定义的策略，如亲和力、应用程序要读取的数据的位置等。
- Controller Manager：是确保集群处于 etcd 中定义的最后状态的 Controller Manager。例如：如果在 etcd 中，一个 deploy 被配置为一个 pod 有 10 个副本，那么是 Controller Manager 会检查集群的当前状态是否与这个状态对应，如果不对应，则会尝试对两者进行协调。
- Kubelet：kubelet 可以看作是运行在 Worker 节点上的 k8s 代理。在每个 worker 节点中，应该有一个 Kubelet 代理在运行。Kubelet 负责实际管理节点内的 pods，这些 pods 是由集群控制器指示的，所以为此 Kubelet 可以按照集群控制器的指示，启动、停止并保持容器和 pods 的运行。
- Kube-proxy：作为代理和负载均衡器。这个组件负责将请求路由到正确的 pod，以及照顾节点的网络部分。
- 容器运行时：容器运行时是 k8s 操作所需的容器执行环境。2016 年加入了 rkt 支持，但自始至终 Docker 都已经是默认的功能和使用。

虽然在标准环境下，k8s 的执行有至少三个节点的要求，但也有在单个节点上运行 k8s 的解决方案。一些例子是：

- Kind：用于执行 Docker 容器的工具，模拟 Kubernetes 集群的运作。它用于教学、开发和测试目的。Kind 不应该用于生产。
- Minikube：用于在本地实现一个只有一个节点的 Kubernetes 集群的工具。广泛用于教学、开发和测试目的。Minikube 不得用于生产。
- MicroK8S：由 Canonical 开发，也就是开发 Ubuntu 的公司。它可以在几个发行版中使用，并可用于生产环境，特别是边缘计算和物联网（物联网）。
- k3s：由 Rancher Labs 开发，它是 MicroK8s 的直接竞争对手，甚至可以运行在 Raspberry Pi 上。

# TBD

- https://draveness.me/ 系列 K8S 相关文章
- https://jimmysong.io/K8s-handbook/concepts/Pod-overview.html
