# K8s 存储

在 Kubernetes 集群中，虽然无状态的服务非常常见，但是在实际的生产中仍然会需要在集群中部署一些有状态的节点，比如一些存储中间件、消息队列等等。在 K8s 中通过 Pod 的概念将一组具有超亲密关系的容器组合到了一起形成了一个服务实例，为了保证一个 Pod 中某一个容器异常退出，被 kubelet 重建拉起旧容器产生的重要数据不丢，以及同一个 Pod 的多个容器可以共享数据，K8s 在 Pod 层面定义了存储卷，不仅能够解决 Container 中文件的临时性问题，也能够让同一个 Pod 中的多个 Container 共享文件。

Docker Volume 是针对容器层面的存储抽象，其 Volume 的生命周期是通过 Docker Engine 来维护的。K8s Volume 则是应用层面的存储抽象，通过 CRI 接口解耦和 Docker Engine 的耦合关系，所以 K8s 的 Volume 的生命周期理所当然由 K8s 来管理，因此 K8s 本身也有自己的 volume plugin 扩展机制。K8s 中的 Volume 主要分为以下几类：

- 本地存储：emptydir/hostpath 等，主要使用 Pod 运行的 node 上的本地存储

- 网络存储：in-tree(内置): awsElasticBlockStore/gcePersistentDisk/nfs 等，存储插件的实现代码是放在 k8s 代码仓库中的；out-of-tree(外置): flexvolume/CSI 等网络存储 inline volume plugins，存储插件单独实现，特别是 CSI 是 Volume 扩展机制的核心发展方向。

- Projected Volume: Secret/ConfigMap/downwardAPI/serviceAccountToken，将 K8s 集群中的一些配置信息以 volume 的方式挂载到 Pod 的容器中，也即应用可以通过 POSIX 接口来访问这些对象中的数据。

- PersistentVolumeClaim 与 PersistentVolume 体系，K8s 中将存储资源与计算资源分开管理的设计体系。

# 卷（Volume）

卷（Volume）其实是一个比较特定的概念，它并不是一个持久化存储，可能会随着 Pod 的删除而删除，常见的卷就包括 EmptyDir、HostPath、ConfigMap 和 Secret，这些卷与所属的 Pod 具有相同的生命周期，它们可以通过如下的方式挂载到 Pod 下面的某一个目录中：

```yml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      volumeMounts:
        - name: cache-volume
          mountPath: /cache
        - name: test-volume
          mountPath: /hostpath
        - name: config-volume
          mountPath: /data/configmap
        - name: special-volume
          mountPath: /data/secret
  volumes:
    - name: cache-volume
      emptyDir: {}
    - name: hostpath-volume
      hostPath:
        path: /data/hostpath
        type: Directory
    - name: config-volume
      configMap:
        name: special-config
    - name: secret-volume
      secret:
        secretName: secret-config
```

需要注意的是，当我们将 ConfigMap 或者 Secret 包装成卷并挂载到某个目录时，我们其实创建了一些新的 Volume，这些 Volume 并不是 Kubernetes 中的对象，它们只存在于当前 Pod 中，随着 Pod 的删除而删除，但是需要注意的是这些临时卷的删除并不会导致相关 ConfigMap 或者 Secret 对象的删除。

从上面我们其实可以看出 Volume 没有办法脱离 Pod 而生存，它与 Pod 拥有完全相同的生命周期，而且它们也不是 Kubernetes 对象，所以 Volume 的主要作用还是用于跨节点或者容器对数据进行同步和共享。

# 持久卷

如果我们希望将数据进行持久化存储，可以引入 PersistentVolume(PV)，以将 Pod 和卷的生命周期分离。我们可以将 PersistentVolume 理解为集群中资源的一种，它与集群中的节点 Node 有些相似，PV 为 Kubernete 集群提供了一个如何提供并且使用存储的抽象，与它一起被引入的另一个对象就是 PersistentVolumeClaim(PVC)，这两个对象之间的关系与节点和 Pod 之间的关系差不多。

因为 PVC 允许用户消耗抽象的存储资源，所以用户需要不同类型、属性和性能的 PV 就是一个比较常见的需求了，在这时我们可以通过 StorageClass 来提供不同种类的 PV 资源，上层用户就可以直接使用系统管理员提供好的存储类型。

## 访问模式

Kubernetes 中的 PV 提供三种不同的访问模式，分别是 `ReadWriteOnce`、`ReadOnlyMany` 和 `ReadWriteMany`，这三种模式的含义和用法我们可以通过它们的名字推测出来：

- `ReadWriteOnce` 表示当前卷可以被一个节点使用读写模式挂载；
- `ReadOnlyMany` 表示当前卷可以被多个节点使用只读模式挂载；
- `ReadWriteMany` 表示当前卷可以被多个节点使用读写模式挂载；

不同的卷插件对于访问模式其实有着不同的支持，AWS 上的 `AWSElasticBlockStore` 和 GCP 上的 `GCEPersistentDisk` 就只支持 `ReadWriteOnce` 方式的挂载，不能同时挂载到多个节点上，但是 `CephFS` 就同时支持这三种访问模式。

## 回收策略

当某个服务使用完某一个卷之后，它们会从 apiserver 中删除 PVC 对象，这时 Kubernetes 就需要对卷进行回收（Reclaim），持久卷也同样包含三种不同的回收策略，这三种回收策略会指导 Kubernetes 选择不同的方式对使用过的卷进行处理。

- 第一种回收策略就是保留（Retain）PV 中的数据，如果希望 PV 能够被重新使用，系统管理员需要删除被使用的 PersistentVolume 对象并手动清除存储和相关存储上的数据。

- 另一种常见的回收策略就是删除（Delete），当 PVC 被使用者删除之后，如果当前卷支持删除的回收策略，那么 PV 和相关的存储会被自动删除，如果当前 PV 上的数据确实不再需要，那么将回收策略设置成 Delete 能够节省手动处理的时间并快速释放无用的资源。

## 存储供应

Kubernetes 集群中包含了很多的 PV 资源，而 PV 资源有两种供应的方式，一种是静态的，另一种是动态的，静态存储供应要求集群的管理员预先创建一定数量的 PV，然后使用者通过 PVC 的方式对 PV 资源的使用进行声明和申请；但是当系统管理员创建的 PV 对象不能满足使用者的需求时，就会进入动态存储供应的逻辑，供应的方式是基于集群中的 StorageClass 对象，当然这种动态供应的方式也可以通过配置进行关闭。
