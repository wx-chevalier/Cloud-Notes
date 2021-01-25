# Rook

Rook 就是一组 Kubernetes 的 Operator，它可以完全控制多种数据存储解决方案（例如 Ceph、EdgeFS、Minio、Cassandra）的部署，管理以及自动恢复。Rook 将分布式存储软件转变为自我管理，自我缩放和自我修复的存储服务。它通过自动化部署，引导、配置、供应、扩展、升级、迁移、灾难恢复、监控和资源管理来实现 Rook 使用基础的云原生容器管理、调度和编排平台提供的功能来履行其职责。

Rook 利用扩展点深入融入云原生环境，为调度、生命周期管理、资源管理、安全性、监控和用户体验提供无缝体验。Rook 最初专注于在 Kubernetes 之上运行 Ceph。Ceph 是一个分布式存储系统，提供文件、数据块和对象存储，可以部署在大型生产集群中。Rook 计划在未来的版本中增加对除 Ceph 之外的其他存储系统以及 Kubernetes 之外的其他云原生环境的支持。

![Rook Architecture](https://s2.ax1x.com/2020/01/14/lqVSeJ.png)

使用 Rook 的其中一个主要好处在于它是通过原生的 Kubernetes 机制和数据存储交互。这就意味着你不再需要通过命令行手动配置 Ceph。
