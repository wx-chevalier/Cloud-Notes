# CGroups

2006 年，Google 的工程师发明了 Linux 控制组，缩写为 CGroups。这是 Linux 内核的一项功能，可隔离和控制用户进程的资源使用情况。这些进程可以放入命名空间，实质上是共享相同资源限制的进程集合。计算机可以有多个命名空间，每个命名空间都具有内核强制执行的资源属性。

我们可以管理每个命名空间的资源分配，以便限制一组进程可以使用的总 CPU，RAM 等的数量。例如，后台日志聚合应用程序可能需要限制其资源，以免意外地压倒它正在记录的实际服务器。虽然不是原始功能，但 Linux 中的 CGroups 最终被重新设计为包含命名空间隔离的功能。命名空间隔离本身并不新鲜，Linux 已经有多种命名空间隔离。一个常见的例子是进程隔离，它将每个单独的进程分开并防止诸如共享内存之类的事情。

CGroups 隔离是一种更高级别的隔离，可确保 CGroups 命名空间中的进程独立于其他命名空间中的进程。

- PID（进程标识符）命名空间：这可确保一个命名空间内的进程不知道其他命名空间中的进程。
- 网络命名空间：隔离网络接口控制器，iptables，路由表和其他低级网络工具。
- 挂载命名空间：已挂载文件系统，因此命名空间的文件系统范围仅限于已挂载的目录。
- 用户名空间：将命名空间内的用户限制为仅限该命名空间，并避免跨命名空间的用户 ID 冲突。

# cgroup-bin

通过工具 cgroup-bin (`sudo apt-get install cgroup-bin`)可以创建 CGroup 并进入该 CGroup 执行命令。

```s
root@host:/home/vagrant# cgcreate -a vagrant -g cpu:cg1
root@host:/home/vagrant# ls /sys/fs/cgroup/cpu/cg1/
cgroup.clone_children  cpu.cfs_period_us  cpu.shares  cpuacct.stat   cpuacct.usage_all     cpuacct.usage_percpu_sys   cpuacct.usage_sys   notify_on_release
cgroup.procs           cpu.cfs_quota_us   cpu.stat    cpuacct.usage  cpuacct.usage_percpu  cpuacct.usage_percpu_user  cpuacct.usage_user  tasks
```

cpu.cfs_period_us 和 cpu.cfs_quota_us，它们分别用来限制该组中的所有进程在单位时间里可以使用的 cpu 时间，这里的 cfs(Completely Fair Scheduler) 是完全公平调度器的意思。cpu.cfs_period_us 是时间周期，默认为 100000，即 100 毫秒。而 cpu.cfs_quota_us 是在时间周期内可以使用的时间，默认为-1 即无限制。cpu.shares 用于限制 cpu 使用的，它用于控制各个组之间的配额。比如组 cg1 的 cpu.shares 为 1024，组 cg2 的 cpu.shares 也是 1024，如果都有进程在运行则它们都可以使用最多 50%的限额。如果 cg2 组内进程比较空闲，那么 cg1 组可以将使用几乎整个 cpu，tasks 存储的是该组里面的进程 ID。

我们先在默认的分组里面运行一个死循环程序 loop.py，因为默认分组/sys/fs/cgroup/cpu/cpu.cfs_period_us 和 cfs_quota_us 是默认值，所以是没有限制 cpu 使用的。可以发现 1 个 cpu us 立马接近 100%了。

```py
# loop.py
while True: pass
```

设置 cg1 组的 cfs_quota_us 位 50000，即表示该组内进程最多使用 50%的 cpu 时间，运行 cgexec 命令进入 cg1 的 cpu 组，然后运行 loop.py，可以发现 cpu us 在 50%以内了，此时也可以在 tasks 文件中看到我们刚刚 cgexec 创建的进程 ID。

```s
root@host:/home/vagrant# echo 50000 > /sys/fs/cgroup/cpu/cg1/cpu.cfs_quota_us
root@host:/home/vagrant# cgexec -g cpu:cg1 /bin/bash
```

Docker 里面要限制内存和 CPU 使用，可以在启动时指定相关参数即可。比如限制 cpu 使用率，加 cpu-period 和 cpu-quota 参数，限制执行的 cpu 核，加--cpuset-cpus 参数。限制内存使用，加--memory 参数。当然，我们可以看到在 /sys/fs/cgroup/cpu/docker/目录下有个以 containerid 为名的分组，该分组下面的 cpu.cfs_period_us 和 cpu.cfs_quota_us 的值就是我们在启动容器时指定的值。

```s
root@host:/home/vagrant# docker run -i -t --cpu-period=100000 --cpu-quota=50000 --memory=512000000 alpine /bin/ash
```

# Capabilities

我们在启动容器时会时常看到这样的参数 `--cap-add=NET_ADMIN`，这是用到了 Linux 的 capability 特性。capability 是为了实现更精细化的权限控制而加入的。我们以前熟知通过设置文件的 SUID 位，这样非 root 用户的可执行文件运行后的 euid 会成为文件的拥有者 ID，比如 passwd 命令运行起来后有 root 权限，有 SUID 权限的可执行文件如果存在漏洞会有安全风险。(查看文件的 capability 的命令为 filecap -a，而查看进程 capability 的命令为 pscap -a，pscap 和 filecap 工具需要安装 libcap-ng-utils 这个包)。

对于 capability，可以看一个简单的例子便于理解。如 Debian 系统中自带的 ping 工具，它是有设置 SUID 位的。这里拷贝 ping 重命名为 anotherping，anotherping 的 SUID 位没有设置，运行会提示权限错误。这里，我们只要将其加上 cap_net_raw 权限即可，不需要设置 SUID 位那么大的权限。

```s
vagrant@host:~$ ls -ls /bin/ping
60 -rwsr-xr-x 1 root root 61240 Nov 10  2016 /bin/ping
vagrant@host:~$ cp /bin/ping anotherping
vagrant@host:~$ ls -ls anotherping
60 -rwxr-xr-x 1 vagrant vagrant 61240 May 19 10:18 anotherping
vagrant@host:~$ ./anotherping -c1 yue.uu.163.com
ping: socket: Operation not permitted
vagrant@host:~$ sudo setcap cap_net_raw+ep ./anotherping
vagrant@host:~$ ./anotherping -c1 yue.uu.163.com
PING yue.uu.163.com (59.111.137.252) 56(84) bytes of data.
64 bytes from 59.111.137.252 (59.111.137.252): icmp_seq=1 ttl=63 time=53.9 ms

--- yue.uu.163.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 53.919/53.919/53.919/0.000 ms
```
