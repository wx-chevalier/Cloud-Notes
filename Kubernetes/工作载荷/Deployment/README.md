# K8s 控制器

Kubernetes 中内建了很多 Controller（控制器），这些相当于一个状态机，用来控制 Pod 的具体状态和行为；Deployment 是一个功能，负责指示 Kubernetes 创建、更新和监控应用实例的健康状况。Deployment 为 Pod 和 ReplicaSet 提供了一个声明式定义方法，用来替代以前的 Replication Controller 来方便的管理应用。只需要在 Deployment 中描述您想要的目标状态是什么，Deployment Controller 就会将 Pod 和 ReplicaSet 的实际状态改变到目标状态。您可以定义一个全新的 Deployment 来创建 ReplicaSet 或者删除已有的 Deployment 并创建一个新的来替换。

Deployment 典型的应用场景是，使用 Deployment 来创建 ReplicaSet。ReplicaSet 在后台创建 Pod 并检查启动状态，看它是成功还是失败。然后，通过更新 Deployment 的 PodTemplateSpec 字段来声明 Pod 的新状态。这会创建一个新的 ReplicaSet。Deployment 会按照控制的速率将 pod 从旧的 ReplicaSet 移动到新的 ReplicaSet 中；如果当前状态不稳定，回滚到之前的 Deployment revision。每次回滚都会更新 Deployment 的 revision。Deployment 还能够完成扩容 Deployment 以满足更高的负载；暂停 Deployment 来应用 PodTemplateSpec 的多个修复，然后恢复上线；根据 Deployment 的状态判断上线是否 hang 住了；清除旧的不必要的 ReplicaSet 等功能。
