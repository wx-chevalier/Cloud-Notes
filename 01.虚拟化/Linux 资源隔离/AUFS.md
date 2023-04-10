# AUFS

联合文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into asingle virtual filesystem)。联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。另外，不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。

Docker 支持的 UnionFS 包括 OverlayFS，AUFS，devicemapper，vfs 以及 btrfs 等，查看 UnionFS 版本可以用 docker info 查看对应输出中的 Storage 项即可，早期的 Docker 版本用 AUFS 和 devicemapper 居多，新版本 Docker 在 Linux 3.18 之后版本基本默认用 OverlayFS。

# OverlayFS

OverlayFS 与早期用过的 AUFS 类似，不过它比 AUFS 更简单，读写性能更好，在 docker-ce18.03 版本中默认用的存储驱动是 overlay2，老版本 overlay 官方已经不推荐使用。它将两个目录 upperdir 和 lowdir 联合挂载到一个 merged 目录，提供统一视图。其中 upperdir 是可读写层，对容器修改写入在该目录中，它也会隐藏 lowerdir 中相同的文件。而 lowdir 是只读层，Docker 镜像在这层。

![OverlayFS](https://i.postimg.cc/FR2m1WZs/image.png)

在看 Docker 镜像和容器存储结构前，可以先简单操作下 OverlayFS 看下基本概念。创建了 lowerdir 和 upperdir 两个目录，然后用 overlayfs 挂载到 merged 目录，这样在 merged 目录可以看到两个目录的所有文件 both.txt 和 only.txt。其中 upperdir 是可读写的，而 lowerdir 只读。通过 merged 目录来操作文件可以发现：

- 读取文件时，如果 upperdir 不存在该文件，则会从 lowerdir 直接读取。

- 修改文件时并不影响 lowerdir 中的文件，因为它是只读的。

- 如果修改的文件在 upperdir 不存在，则会从 lowerdir 拷贝到 upperdir，然后在 upperdir 里面修改该文件，并不影响 lowerdir 目录的文件。

- 删除文件则是将 upperdir 中将对应文件设置成了 c 类型，即字符设备类型来隐藏已经删除的文件(与 AUFS 创建一个 whiteout 文件略有不同)。

```s
root@host:/home/vagrant/overlaytest# tree -a
.
|-- lowerdir
|   |-- both.txt
|   `-- only.txt
|-- merged
|-- upperdir
|   `-- both.txt
`-- workdir
    `-- work

5 directories, 3 files
root@host:/home/vagrant/overlaytest# mount -t overlay overlay -olowerdir=./lowerdir,upperdir=./upperdir,workdir=./workdir ./merged
root@host:/home/vagrant/overlaytest# tree
.
|-- lowerdir
|   |-- both.txt
|   `-- only.txt
|-- merged
|   |-- both.txt
|   `-- only.txt
|-- upperdir
|   `-- both.txt
`-- workdir
    `-- work

5 directories, 5 files
root@host:/home/vagrant/overlaytest# tree -a
.
|-- lowerdir
|   |-- both.txt
|   `-- only.txt
|-- merged
|   |-- both.txt
|   `-- only.txt
|-- upperdir
|   `-- both.txt
`-- workdir
    `-- work

5 directories, 5 files
root@host:/home/vagrant/overlaytest# echo "modified both" > merged/both.txt
root@host:/home/vagrant/overlaytest# cat upperdir/both.txt
modified both
root@host:/home/vagrant/overlaytest# cat lowerdir/both.txt
lower both.txt
root@host:/home/vagrant/overlaytest# echo "modified only" > merged/only.txt
root@host:/home/vagrant/overlaytest# tree
.
|-- lowerdir
|   |-- both.txt
|   `-- only.txt
|-- merged
|   |-- both.txt
|   `-- only.txt
|-- upperdir
|   |-- both.txt
|   `-- only.txt
`-- workdir
    `-- work

5 directories, 6 files
root@host:/home/vagrant/overlaytest# cat upperdir/only.txt
modified only
root@host:/home/vagrant/overlaytest# cat lowerdir/only.txt
lower only.txt
root@host:/home/vagrant/overlaytest# tree -a
.
|-- lowerdir
|   |-- both.txt
|   `-- only.txt
|-- merged
|   |-- both.txt
|   `-- only.txt
|-- upperdir
|   |-- both.txt
|   `-- only.txt
`-- workdir
    `-- work

5 directories, 6 files
root@host:/home/vagrant/overlaytest# rm merged/both.txt
root@host:/home/vagrant/overlaytest# tree -a
.
|-- lowerdir
|   |-- both.txt
|   `-- only.txt
|-- merged
|   `-- only.txt
|-- upperdir
|   |-- both.txt
|   `-- only.txt
`-- workdir
    `-- work
root@host:/home/vagrant/overlaytest# ls -ls upperdir/both.txt
0 c--------- 1 root root 0, 0 May 19 02:31 upperdir/both.txt
```

回到 Docker 里面，我们拉取一个 Nginx 镜像，有三层镜像，可以看到在 overlay2 对应每一层都有个目录(注意，这个目录名跟镜像层名从 docker1.10 版本后名字已经不对应了)，另外的 l 目录是指向镜像层的软链接。最底层存储的是基础镜像 debian/alpine，上一层是安装了 nginx 增加的可执行文件和配置文件，而最上层是链接/dev/stdout 到 nginx 日志文件。而每个子目录下面的 diff 目录用于存储镜像内容，work 目录是 OverlayFS 内部使用的，而 link 文件存储的是该镜像层对应的短名称，lower 文件存储的是下一层的短名称。

```s
root@stretch:/home/vagrant# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
f2aa67a397c4: Pull complete
3c091c23e29d: Pull complete
4a99993b8636: Pull complete
Digest: sha256:0fb320e2a1b1620b4905facb3447e3d84ad36da0b2c8aa8fe3a5a81d1187b884
Status: Downloaded newer image for nginx:latest

root@stretch:/home/vagrant# ls -ls /var/lib/docker/overlay2/
total 16
4 drwx------ 4 root root 4096 May 19 04:17 09495e5085bced25e8017f558147f82e61b012a8f632a0b6aac363462b1db8b0
4 drwx------ 3 root root 4096 May 19 04:17 8af95287a343b26e9c3dd679258773880e7bdbbe914198ba63a8ed1b4c5f5554
4 drwx------ 4 root root 4096 May 19 04:17 f311565fe9436eb8606f846e1f73f38287841773e8d041933a41259fe6f96afe
4 drwx------ 2 root root 4096 May 19 04:17 l

root@stretch:/var/lib/docker/overlay2# ls  09495e5085bced25e8017f558147f82e61b012a8f632a0b6aac363462b1db8b0/
diff  link  lower  work
```

从我们示例可以看到，三层中 f311 是最顶层，下面分别是 0949 和 8af9 这两层。

```s
root@stretch:/var/lib/docker/overlay2# cat f311565fe9436eb8606f846e1f73f38287841773e8d041933a41259fe6f96afe/lower
l/7B2WM6DC226TCJU6QHJ4ABKRI6:l/4FHO2G5SWWRIX44IFDHU62Z7X2
root@stretch:/var/lib/docker/overlay2# cat 09495e5085bced25e8017f558147f82e61b012a8f632a0b6aac363462b1db8b0/lower
l/4FHO2G5SWWRIX44IFDHU62Z7X2
root@stretch:/var/lib/docker/overlay2# cat 8af95287a343b26e9c3dd679258773880e7bdbbe914198ba63a8ed1b4c5f5554/link
4FHO2G5SWWRIX44IFDHU62Z7X2
```

此时我们启动一个 Nginx 容器，可以看到 overlay2 目录多了两个目录，多出来的就是容器层的目录和只读的容器 init 层。容器目录下面的 merged 就是我们前面提到的联合挂载目录了，而 lowdir 则是它下层目录。而容器 init 层用来存储与这个容器内环境相关的内容，如 /etc/hosts 和 /etc/resolv.conf 文件，它居于其他镜像层之上，容器层之下。

```s
root@stretch:/var/lib/docker/overlay2# docker run -idt --name nginx nginx
01a873eeba41f00a5a3deb083adf5ed892c55b4680fbc2f1880e282195d3087b

root@stretch:/var/lib/docker/overlay2# ls -ls
4 drwx------ 4 root root 4096 May 19 04:17 09495e5085bced25e8017f558147f82e61b012a8f632a0b6aac363462b1db8b0
4 drwx------ 5 root root 4096 May 19 09:11 11b7579a1f1775ad71fe0f0f45fcb74c241fce319f5125b1b92cb442385065b1
4 drwx------ 4 root root 4096 May 19 09:11 11b7579a1f1775ad71fe0f0f45fcb74c241fce319f5125b1b92cb442385065b1-init
4 drwx------ 3 root root 4096 May 19 04:17 8af95287a343b26e9c3dd679258773880e7bdbbe914198ba63a8ed1b4c5f5554
4 drwx------ 4 root root 4096 May 19 04:17 f311565fe9436eb8606f846e1f73f38287841773e8d041933a41259fe6f96afe
4 drwx------ 2 root root 4096 May 19 09:11 l

root@stretch:/home/vagrant# ls -ls /var/lib/docker/overlay2/11b7579a1f1775ad71fe0f0f45fcb74c241fce319f5125b1b92cb442385065b1/
4 drwxr-xr-x 4 root root 4096 May 19 09:11 diff
4 -rw-r--r-- 1 root root   26 May 19 09:11 link
4 -rw-r--r-- 1 root root  115 May 19 09:11 lower
4 drwxr-xr-x 1 root root 4096 May 19 09:11 merged
4 drwx------ 3 root root 4096 May 19 09:11 work

root@stretch:/var/lib/docker/overlay2# ls  11b7579a1f1775ad71fe0f0f45fcb74c241fce319f5125b1b92cb442385065b1/merged/
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

root@stretch:/var/lib/docker/overlay2# ls 11b7579a1f1775ad71fe0f0f45fcb74c241fce319f5125b1b92cb442385065b1/diff/
run  var
```

如果我们在容器中修改文件，则会反映到容器层的 merged 目录相关文件，容器层的 diff 目录相当于 upperdir，其他层是 lowerdir。如果之前容器层 diff 目录不存在该文件，则会拷贝该文件到 diff 目录并修改。读取文件时，如果 upperdir 目录找不到，则会直接从下层的镜像层中读取。

# 写时复制

容器运行时的只读模板。每一个镜像由一系列的层 (layers) 组成，层是由 Dockerfile 指定。copy on write 写时复制。容器是由镜像所创建，会根据多层文件系统构建一个镜像栈，只有栈的最顶层是读写层。如果发生对只读层的写操作时会将该文件复制到读写层，并隐藏只读层的文件。

# Links

- https://jvns.ca/blog/2019/11/18/how-containers-work--overlayfs/ How containers work: overlayfs
