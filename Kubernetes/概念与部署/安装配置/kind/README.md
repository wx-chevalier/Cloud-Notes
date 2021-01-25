# kind

kind 是一个用于在 Docker 容器节点中运行本地 Kubernetes 集群的工具。通过与 kubetest 集成，kind 使 Kubernetes 中的端到端测试变得很简单 。我们已经借助 kind 创建临时性的 Kubernetes 集群，在持续集成（Continuous Integration，CI）管道里测试 Kubernetes 中的资源，例如控制器和自定义资源（Custom Resource Definitions，CRDs）。
