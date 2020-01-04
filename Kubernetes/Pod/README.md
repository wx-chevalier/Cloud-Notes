# Pod

Pod 是 Kubernetes 可以部署和管理的最小的可部署单元。换句话说，如果您需要在 Kubernetes 中运行单个容器，则需要为该容器创建一个 Pod。同时，如果这些容器相对紧密地耦合，则 Pod 可以包含多个容器。在 Pod 里面，Container 之间可以共享网络（IP/Port）、共享存储（Volume）、共享 Hostname。另外，Pod 可以理解成一个逻辑主机，它与非容器领域的物理主机或者 VM 有着类似的行为。在同一个 Pod 运行的进程就像在同一物理主机或 VM 上运行的进程一样，只是这些进程被单独的放到单个 container 内。

# 链接

- https://mp.weixin.qq.com/s/Vrgff_qfKWPFmRPb_LVMmQ
