# Helm 的使用

Helm 使用称为 Chart 的包装格式。Charts 是描述相关的一组 Kubernetes 资源的文件集合。单个 Charts 可能用于部署简单的东西，比如 memcached pod，或者一些复杂的东西，比如完整的具有 HTTP 服务，数据库，缓存等的 Web 应用程序堆栈。

Chart 通过创建为特定目录树的文件，将它们打包到版本化的压缩包，然后进行部署。一个 Chart 是一个 Helm 包。它包含在 Kubernetes 集群内部运行应用程序，工具或服务所需的所有资源定义。把它想像为一个自制软件，一个 Apt dpkg 或一个 Yum RPM 文件的 Kubernetes 环境里面的等价物。一个 Repository 是 Charts 收集和共享的地方。它就像 Perl 的 CPAN archive 或 Fedora 软件包 repoFedora Package Database。

一个 Release 是处于 Kubernetes 集群中运行的 Chart 的一个实例。一个 Charts 通常可以多次安装到同一个群集中。每次安装时，都会创建一个新 release 。比如像一个 MySQL Charts。如果希望在群集中运行两个数据库，则可以安装该 Charts 两次。每个都有自己的 release，每个 release 都有自己的 release name。

有了这些概念，我们现在可以这样解释 Helm：Helm 将 Chartss 安装到 Kubernetes 中，每个安装创建一个新 release 。要找到新的 Charts，可以搜索 Helm Chartss 存储库 repositories。

# 命令详解

## 'helm search': 查找 Charts

首次安装 Helm 时，它已预配置为使用官方 Kubernetes Charts 存储库 repo。该 repo 包含许多精心设计和维护的 Chartss。此 Chartss repo 默认以 stable 命名。可以通过运行 helm search 查看有哪些 Chartss 可用：

```s
$ helm search
NAME                     VERSION     DESCRIPTION
stable/drupal       0.3.2       One of the most versatile open source content m...
stable/jenkins      0.1.0       A Jenkins Helm Charts for Kubernetes.
stable/mariadb      0.5.1       Chart for MariaDB
stable/mysql        0.1.0       Chart for MySQL
...
```

如果没有使用过滤条件，helm search 显示所有可用的 Chartss。可以通过使用过滤条件进行搜索来缩小搜索的结果范围：

```s
$ helm search mysql
NAME                   VERSION    DESCRIPTION
stable/mysql      0.1.0      Chart for MySQL
stable/mariadb    0.5.1      Chart for MariaDB
```

现在只会看到与过滤条件匹配的结果。为什么 mariadb 在列表中？因为它的包描述与 MySQL 相关。我们可以使用 helm inspect Charts 到这个：

```s
$ helm inspect stable/mariadb
Fetched stable/mariadb to mariadb-0.5.1.tgz
description: Chart for MariaDB
engine: gotpl
home: https://mariadb.org
keywords:
- mariadb
- mysql
- database
- sql
...
```

## 'helm install'：安装一个软件包

要安装新的软件包，请使用该 helm install 命令。最简单的方法，它只需要一个参数：Charts 的名称。

```s
$ helm install stable/mariadb
Fetched stable/mariadb-0.3.0 to /Users/mattbutcher/Code/Go/src/k8s.io/helm/mariadb-0.3.0.tgz
NAME: happy-panda
LAST DEPLOYED: Wed Sep 28 12:32:28 2016
NAMESPACE: default
STATUS: DEPLOYED

Resources:
==> extensions/Deployment
NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
happy-panda-mariadb   1         0         0            0           1s

==> v1/Secret
NAME                     TYPE      DATA      AGE
happy-panda-mariadb   Opaque    2         1s

==> v1/Service
NAME                     CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
happy-panda-mariadb   10.0.0.70    <none>        3306/TCP   1s


Notes:
MariaDB can be accessed via port 3306 on the following DNS name from within your cluster:
happy-panda-mariadb.default.svc.cluster.local

To connect to your database run the following command:

   kubectl run happy-panda-mariadb-client --rm --tty -i --image bitnami/mariadb --command -- mysql -h happy-panda-mariadb
```

现在 mariadb Charts 已安装，请注意，安装 Charts 会创建一个新 release 对象。上面的 release 被命名 为 happy-panda。（如果你想使用你自己的 release 名称，只需使用 --name 参数 配合 helm install。）在安装过程中，helm 客户端将打印有关创建哪些资源的有用信息，release 的状态以及是否可以或应该采取其他的配置步骤。

Helm 不会一直等到所有资源都运行才退出。许多 Chartss 需要大小超过 600M 的 Docker 镜像，因此可能需要很长时间才能安装到群集中。

### helm status

要跟踪 release 状态或重新读取配置信息，可以使用 helm status：

```s
$ helm status happy-panda
Last Deployed: Wed Sep 28 12:32:28 2016
Namespace: default
Status: DEPLOYED

Resources:
==> v1/Service
NAME                     CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
happy-panda-mariadb   10.0.0.70    <none>        3306/TCP   4m

==> extensions/Deployment
NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
happy-panda-mariadb   1         1         1            1           4m

==> v1/Secret
NAME                     TYPE      DATA      AGE
happy-panda-mariadb   Opaque    2         4m


Notes:
MariaDB can be accessed via port 3306 on the following DNS name from within your cluster:
happy-panda-mariadb.default.svc.cluster.local

To connect to your database run the following command:

   kubectl run happy-panda-mariadb-client --rm --tty -i --image bitnami/mariadb --command -- mysql -h happy-panda-mariadb
```
