# Helm

Helm 是 K8s 的包管理器，Helm chart 用于帮助你定义、安装、更新 K8s 应用，可以处理那些最复杂那种部署情况。

在 K8s 中，容器即进程，它解决了如何部署和运行应用的问题。对于任何一个部署在 K8s 的应用而言，通常都可以由几个固定的部分组成：Ingress、Service、Deployment 等。直接使用 K8s 原生的 YAML 定义服务，虽然能一定程度上简化应用的部署，但是对于大部分研发人员来说编写和使用 YAML 依然是一件相对痛苦的事情。Helm 应运而生，Helm 作为 K8s 下的包管理工具，对原生服务定义过程进行了增强，通过模板化，参数化的形式大大简化用户部署 K8s 应用的复杂度。

可以将 Helm 理解为 K8s 的包管理工具，可以方便地发现、共享和使用为 K8s 构建的应用，有点类似于 Ubuntu 的 APT 和 CentOS 中的 yum。Helm chart 是用来封装 K8s 原生应用程序的 yaml 文件，可以在你部署应用的时候自定义应用程序的一些 metadata，便与应用程序的分发。它包含几个基本概念：

- Chart：一个 Helm 包，其中包含了运行一个应用所需要的镜像、依赖和资源定义等，还可能包含 K8s 集群中的服务定义，类似 Homebrew 中的 formula，APT 的 dpkg 或者 Yum 的 rpm 文件，

- Release: 在 K8s 集群上运行的 Chart 的一个实例。在同一个集群上，一个 Chart 可以安装很多次。每次安装都会创建一个新的 release。例如一个 MySQL Chart，如果想在服务器上运行两个数据库，就可以把这个 Chart 安装两次。每次安装都会生成自己的 Release，会有自己的 Release 名称。

- Repository：用于发布和存储 Chart 的仓库。

# Helm 组件

Helm 采用客户端/服务器架构，有如下组件组成：

- Helm CLI 是 Helm 客户端，可以在本地执行。

- Tiller 是服务器端组件，在 K8s 群集上运行，并管理 K8s 应用程序的生命周期。

- Repository 是 Chart 仓库，Helm 客户端通过 HTTP 协议来访问仓库中 Chart 的索引文件和压缩包。

# 命令控制

Helm 是由 Deis 发起的一个开源工具，有助于简化部署和管理 Kubernetes 应用。在本章的实践中，我们也会使用 Helm 来简化很多应用的安装操作。

![](https://i.postimg.cc/HkrFs1Cb/image.png)

在 Linux 中可以使用 Snap 安装 Heml：

```sh
$ sudo snap install helm --classic

# 通过键入如下命令，在 Kubernetes 群集上安装 Tiller
$ helm init --upgrade
```

在缺省配置下，Helm 会利用 "gcr.io/kubernetes-helm/tiller" 镜像在 Kubernetes 集群上安装配置 Tiller；并且利用 "https://kubernetes-charts.storage.googleapis.com" 作为缺省的 stable repository 的地址。由于在国内可能无法访问 "gcr.io", "storage.googleapis.com" 等域名，阿里云容器服务为此提供了镜像站点。请执行如下命令利用阿里云的镜像来配置 Helm：

```sh
$ helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.5.1 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

# 删除默认的源
$ helm repo remove stable

# 设置 Helm 命令自动补全
$ source <(helm completion zsh)
$ source <(helm completion bash)

# 增加新的国内镜像源
$ helm repo add stable https://burdenbear.github.io/kube-charts-mirror/
$ helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

# 查看 Helm 源添加情况
$ helm repo list
```

Helm 的常见命令如下：

```sh
# 查看在存储库中可用的所有 Helm Charts
$ helm search

# 删除并更新源 or https://burdenbear.github.io/kube-charts-mirror/
$ helm repo remove stable && helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
# 更新 Charts 列表以获取最新版本
$ helm repo update

# 部署某个本地 Chart，指定命名空间与额外的配置文件
$ helm install --namespace ufc --name ufc-dev -f ./deployment/ufc/dev-values.yaml ./charts/ufc/

# 查看某个 Chart 的变量
$ helm inspect values stable/mysql

# 查看在群集上安装的 Charts 列表
$ helm list

# 调试某个配置
$ helm install --debug --dry-run ./mychart

# 校验某个配置
$ docker run -it --rm --name ct --volume $(pwd):/data quay.io/helmpack/chart-testing:v2.3.0 sh -c "ct lint --all --debug --chart-dirs /data/"

# 更新某个配置
$ helm upgrade my-release stable/external-dns
$ helm upgrade -f panda.yaml happy-panda stable/mariadb

# 删除某个 Charts 的部署
$ helm del --purge wordpress-test

# 为 Tiller 部署添加授权
$ kubectl create serviceaccount --namespace kube-system tiller
$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
$ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```
