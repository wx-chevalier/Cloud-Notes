# Focker in Go

这里我们参考 [Gocker](https://unixism.net/2020/06/containers-the-hard-way-gocker-a-mini-docker-written-in-go/) 的实现，实现我们的 Focker。

![Gocker Screenshot](https://s1.ax1x.com/2020/06/18/NeRI7q.png)

用 Gocker 创建的容器拥有自己的以下命名空间（请参见`run.go`和`network.go`）：

- File system (via `chroot`)
- PID
- IPC
- UTS (hostname)
- Mount
- Network

当创建 cgroups 来限制以下内容时，除非您在 `gocker run` 命令中指定了 `--mem`，`--cpus` 或 `--pids` 选项，否则接续器将使用无限的资源。这些标志分别限制了容器可以使用的最大 RAM，CPU 内核和 PID。

- Number of CPU cores
- RAM
- Number of PIDs (to limit processes)

# Namespaces basics

# Links

- https://unixism.net/2020/06/containers-the-hard-way-gocker-a-mini-docker-written-in-go/
