# 容器技术与资源隔离

容器是一种轻量级的虚拟化技术，它不需要模拟硬件创建虚拟机。在 Linux 系统里面，使用到 Linux kernel 的 CGroups，Namespace(ipc，network，user，pid，mount），capability 等用于隔离运行环境和资源限制的技术。容器技术早就出现。例如 Solaris Zones 和 BSD jails 就是非 Linux 操作系统上的容器，而用于 Linux 的容器技术也有很多如 Linux-Vserver、OpenVZ 和 FreeVPS。虽然这些技术都已经成熟，但是这些解决方案还没有将它们的容器支持集成到主流 Linux 内核。总的来说，容器不等同于 Docker，容器更不是虚拟机。

![Docker & Linux Internals](https://i.postimg.cc/d3x9b3NT/image.png)

Linux CGroups 为一种名为 Linux 容器（LXC）的技术铺平了道路。LXC 实际上是我们今天所知的第一个实现容器的主要实现，利用 cgroup 和命名空间隔离来创建具有独立进程和网络空间的虚拟环境。从某种意义上说，这允许独立和隔离的用户空间。容器的概念直接来自 LXC。事实上，早期版本的 Docker 直接构建在 LXC 之上。Docker 最初目标是做一个特殊的 LXC 的开源系统，最后慢慢演变为它自己的一套容器运行时环境。Docker 基于 Linux kernel 的 CGroups，Namespace，UnionFileSystem 等技术封装成一种自定义的容器格式，用于提供一整套虚拟运行环境。毫无疑问，近些年来 Docker 已经成为了容器技术的代名词。

![Docker 与虚拟机](https://i.postimg.cc/BZkHfrQD/image.png)

# After `docker run`

通过下面命令运行一个 debian 容器，attach 到一个本机的命令行并运行/bin/bash。

```undefined
docker run -i -t debian /bin/bash
```

这个命令背后都做了什么？

- 1.如果本机没有 debian 镜像，则会从你配置的 Registry 里面拉取一个 debian 的 latest 版本的镜像，跟你运行了`docker pull debian`效果一样。
- 2.创建容器。跟运行`docker create`一样。
- 3.给容器分配一个读写文件系统作为该容器的`final layer`，容器可以在它的文件系统创建和修改文件。
- 4.Docker 为容器创建了一套网络接口，给容器分配一个 ip。默认情况下，容器可以通过默认网络连通到外部网络。
- 5.Docker 启动容器并执行 `/bin/bash`。因为启动时指定了`-i -t`参数，容器是以交互模式运行且 attach 到本地终端，我们可以在终端上输入命令并看到输出。
- 6.运行 exit 可以退出容器，但是此时容器并没有被删除，我们可以再次运行它或者删除它。

可以发现，容器的内核版本是跟宿主机一样的，不同的是容器的主机名是独立的，它默认用容器 ID 做主机名。我们运行`ps -ef`可以发现容器进程是隔离的，容器里面看不到宿主机的进程，而且它自己有 PID 为 1 的进程。此外，网络也是隔离的，它有独立于宿主机的 IP。文件系统也是隔离的，容器有自己的系统和软件目录，修改容器内的文件并不影响宿主机对应目录的文件。

```ruby
root@stretch:/home/vagrant# uname -r
4.9.0-6-amd64
root@stretch:/home/vagrant# docker run -it --name demo alpine /bin/ash
/ # uname -r   ## 容器内
4.9.0-6-amd64

/ # ps -ef
PID   USER     TIME   COMMAND
    1 root       0:00 /bin/ash
    7 root       0:00 ps -ef

/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

这些隔离机制并不是 Docker 新开发的技术，而是依托 Linux kernel 以及一些已有的技术实现的，主要包括：

- 隔离性，Linux Namespaces(Linux2.6.24 后引入)：命名空间用于进程(PID)、网络(NET)、挂载点(MNT)、UTS、IPC 等隔离。容器内的应用只能在自己的命名空间中运行而且不会访问到命名空间之外。

- 控制组，Linux Control Groups(CGroups)：用于限制容器使用的资源，包括内存，CPU 等。使应用隔离运行的关键是让它们只使用你想要的资源。这样可以确保在机器上运行的容器都是良民(good multi-tenant citizens)。群组控制允许 Docker 分享或者限制容器使用硬件资源。

- 便携性，Union File Systems：UnionFS 把多个目录结合成一个目录，对外使用，最上层目录为读写层(通常只有 1 个)，下面可以有一个或多个只读层，见容器和镜像分层图。Docker 支持 OverlayFS，AUFS、DeviceMapper、btrfs 等联合文件系统，支持将不同目录挂载到同一个虚拟文件系统下的文件系统。

- Container Format: Docker Engine 组合 Namespaces，CGroups 以及 UnionFS 包装为一个容器格式，默认格式为 libcontainer，后续可能会加入 BSD Jails 或 Solaris Zones 容器格式的支持。

# Links

- https://unit42.paloaltonetworks.com/making-containers-more-isolated-an-overview-of-sandboxed-container-technologies

- https://jiajially.gitbooks.io/dockerguide/content/dockerCoreNS.html 提取其中的命令与原理解析
