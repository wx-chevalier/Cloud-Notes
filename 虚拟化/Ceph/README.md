# Ceph

Ceph 是 linux 系统中一个分布式文件系统，能够在维护 POSIX 兼容性的同时加入了复制和容错功能，由 Sage Weil 在 University of California, SantaCruz（UCSC）实施。Ceph 内部实现了分布式数据对象存储，对外可以提供文件系统、对象、块设备的访问方式，实现了统一存储平台。Ceph 社区最新版本是 14，而 Ceph 12 是市面用的最广的稳定版本。

目前，Ceph 主要有三种企业级应用场景：

- IOPS 密集型：这种类型的场景通常是支撑在虚拟化/私有云上运行数据库。如在 OpenStack 上运行 Mysql、MariaDB 或 PostgreSQL 等。IOPS 密集型场景对磁盘的性能要求较高，最好使用全闪架构。如果使用混合架构，机械盘转速需要 1.2 万，并使用高速盘存储频繁写操作的日志或元数据。
- 高吞吐量型：这种类型的应用场景主要是大块数据传输，如图像、视频、音频文件等。高吞吐量型磁盘的要求没有 IOPS 密集型高，但需要配置较高的网络。同时也需要配置 SSD 来处理写日志。
- 高容量型：这种场景主要用于存储归档、离线数据。它对磁盘的容量要求高，对性能无过多要求。写日志也可以存储在 HDD 上。

# Ceph 架构

Ceph 由储存管理器(Object storage cluster 对象存储集群，即：Osd 守护进程)、,集群监视器(Ceph Monitor)和元数据服务器(Metadata server cluster, mds)构成。其中，元数据服务器 MDS 仅仅在客户端(数据用户 Client)通过文件系统方式使用 Ceph 时有用。当客户端通过块设备或对象存储使用 CEPH 时，可以没有 MDS。一个 Ceph 储存集群，由一系列的节点(具备 CPU 和 MEM 的计算机)、储存设备和传输网络构成。

![Ceph 架构](https://s2.ax1x.com/2020/01/14/lqE5LQ.png)

Ceph 存储集群由三类守护进程组成：OSD，Monitor 和 Manager。

- OSD：OSD 是 Ceph 存储数据的空间，通常一个 HDD 是一个 OSD，并且不建议做 RAID（独立硬盘冗余阵列）。每个 OSD 有一个 OSD 守护进程。Ceph OSD 利用 Ceph 节点的 CPU、内存和网络资源来执行数据复制、纠删码、数据恢复、监控和报告功能。

- Monitor：Monitor 负责维护 Ceph 存储集群，主要是集群中数据的主副本以及存储集群的当前状态。注意，多个 Monitor 的信息需要强一致性，因此要求 Monitor 节点之间的系统时间是一致的，并且网络延时要低。

- Manager：Manager 是 Ceph 12.8 中的新功能，它维护放置组（PG）、进程元数据和主机元数据的详细信息。这部分功能此前由 Monitor 完成（其目的是提高 Ceph 集群的性能）。Manager 可以处理只读 Ceph CLI 查询请求，例如放置组统计信息等。此外，Manager 还提供 RESTful 监控 API。

如果要使用 Ceph 文件系统和对象接口，Ceph 集群还需要如下节点：

- 元数据服务器（Metadata Server，简称 MDS）：每个 MDS 节点运行 MDS 守护程序（Ceph-mds）管理与 Ceph 文件系统（CephFS）上存储的文件相关的元数据。
- 对象网关：Ceph 对象网关节点上运行 Ceph RADOS 网关守护程序（Ceph-radosgw）。它是一个构建在 librados 之上的对象存储接口，也是一个为应用程序提供 Ceph 存储集群的 RESTful 网关。Ceph 对象网关支持两个接口：S3 和 OpenStack Swift。
