# 容器虚拟化

**Machine-level virtualization**, such as [KVM](https://www.linux-kvm.org/) and [Xen](https://www.xenproject.org/), exposes virtualized hardware to a guest kernel via a Virtual Machine Monitor (VMM). This virtualized hardware is generally enlightened (paravirtualized) and additional mechanisms can be used to improve the visibility between the guest and host (e.g. balloon drivers, paravirtualized spinlocks). Running containers in distinct virtual machines can provide great isolation, compatibility and performance (though nested virtualization may bring challenges in this area), but for containers it often requires additional proxies and agents, and may require a larger resource footprint and slower start-up times.

[![Machine-level virtualization](https://github.com/google/gvisor/raw/master/g3doc/Machine-Virtualization.png)](https://github.com/google/gvisor/blob/master/g3doc/Machine-Virtualization.png)

**Rule-based execution**, such as [seccomp](https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt), [SELinux](https://selinuxproject.org/) and [AppArmor](https://wiki.ubuntu.com/AppArmor), allows the specification of a fine-grained security policy for an application or container. These schemes typically rely on hooks implemented inside the host kernel to enforce the rules. If the surface can be made small enough (i.e. a sufficiently complete policy defined), then this is an excellent way to sandbox applications and maintain native performance. However, in practice it can be extremely difficult (if not impossible) to reliably define a policy for arbitrary, previously unknown applications, making this approach challenging to apply universally.

[![Rule-based execution](https://github.com/google/gvisor/raw/master/g3doc/Rule-Based-Execution.png)](https://github.com/google/gvisor/blob/master/g3doc/Rule-Based-Execution.png)

Rule-based execution is often combined with additional layers for defense-in-depth.

**gVisor** provides a third isolation mechanism, distinct from those mentioned above.

gVisor intercepts application system calls and acts as the guest kernel, without the need for translation through virtualized hardware. gVisor may be thought of as either a merged guest kernel and VMM, or as seccomp on steroids. This architecture allows it to provide a flexible resource footprint (i.e. one based on threads and memory mappings, not fixed guest physical resources) while also lowering the fixed costs of virtualization. However, this comes at the price of reduced application compatibility and higher per-system call overhead.

[![gVisor](https://github.com/google/gvisor/raw/master/g3doc/Layers.png)](https://github.com/google/gvisor/blob/master/g3doc/Layers.png)

On top of this, gVisor employs rule-based execution to provide defense-in-depth (details below).

gVisor's approach is similar to [User Mode Linux (UML)](http://user-mode-linux.sourceforge.net/), although UML virtualizes hardware internally and thus provides a fixed resource footprint.

与虚拟机相比，容器更轻量且速度更快，因为它利用了 Linux 底层操作系统在隔离的环境中运行。虚拟机的 hypervisor 创建了一个非常牢固的边界，以防止应用程序突破它，而容器的边界不那么强大。另一个区别是，由于 Namespace 和 Cgroups 功能仅在 Linux 上可用，因此容器无法在其他操作系统上运行；Docker 实际上使用了一个技巧，并在非 Linux 操作系统上安装 Linux 虚拟机，然后在虚拟机内运行容器。

虽然大多数 IT 行业正在采用基于容器的基础架构（云原生解决方案），但必须了解该技术的局限性。传统容器（如 Docker，Linux Containers（LXC）和 Rocket（rkt））并不是真正的沙箱，因为它们共享主机操作系统内核。它们具有资源效率，但攻击面和破坏的潜在影响仍然很大，特别是在多租户云环境中，共同定位属于不同客户的容器。

当主机操作系统为每个容器创建虚拟化用户空间时，问题的根源是容器之间的弱分离。一直致力于设计真正的沙盒容器的研究和开发。大多数解决方案重新构建容器之间的边界以加强隔离。譬如来自 IBM，Google，Amazon 和 OpenStack 的四个独特项目，这些项目使用不同的技术来实现相同的目标，为容器创建更强的隔离。

- IBM Nabla 在 Unikernels 之上构建容器
- Google gVisor 创建了一个用于运行容器的专用客户机内核
- Amazon Firecracker 是一个用于沙盒应用程序的极轻量级管理程序
- OpenStack 将容器放置在针对容器编排平台优化的专用 VM 中
