# Overlayfs

容器运行时的只读模板。每一个镜像由一系列的层 (layers) 组成，层是由 Dockerfile 指定。copy on write 写时复制。容器是由镜像所创建，会根据多层文件系统构建一个镜像栈，只有栈的最顶层是读写层。如果发生对只读层的写操作时会将该文件复制到读写层，并隐藏只读层的文件。

联合文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into asingle virtual filesystem)。联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。另外，不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。

Docker 中使用的 AUFS（Another UnionFS）就是一种联合文件系统。AUFS 支持为每一个成员目录（类似 Git 的分支）设定只读（readonly）、读写（readwrite）和写出（whiteout-able）权限, 同时 AUFS 里有一个类似分层的概念, 对只读权限的分支可以逻辑上进行增量地修改(不影响只读部分的)。Docker 目前支持的联合文件系统种类包括 AUFS, btrfs, vfs 和 DeviceMapper

# 链接

- https://jvns.ca/blog/2019/11/18/how-containers-work--overlayfs/ How containers work: overlayfs
