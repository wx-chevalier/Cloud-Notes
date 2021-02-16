# ConfigMap

应用部署的一个最佳实践是将应用所需的配置信息与程序进行分离，这样可以使得应用程序被更好地复用，通过不同的配置也能实现更灵活的功能。将应用打包为容器镜像后，可以通过环境变量或者外挂文件的方式在创建容器时进行配置注入，但在大规模容器集群的环境中，对多个容器进行不同的配置将变得非常复杂。ConfigMap 即是用来存储配置文件的 K8s 资源对象，所有的配置内容都存储在 etcd 中。

![ConfigMap 示意图](https://matthewpalmer.net/kubernetes-app-developer/articles/configmap-diagram.gif)

ConfigMap 供容器使用的典型用法如下：

- 生成为容器内的环境变量；
- 设置容器启动命令的启动参数（需设置为环境变量）；
- 以 Volume 的形式挂载为容器内部的文件或目录。

# TBD

- https://learning.oreilly.com/library/view/kubernetes-for-developers/9781788834759/961e9251-74a4-4e47-8a48-9c02f3500bbb.xhtml
- https://blog.csdn.net/liukuan73/article/details/79492374
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
- https://www.jianshu.com/p/d834bca35c18
