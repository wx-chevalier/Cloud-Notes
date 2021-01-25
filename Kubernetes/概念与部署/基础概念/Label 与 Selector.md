# Label

元数据围绕应用（Application）的概念进行组织，Kubernetes 不是平台即服务（PaaS），没有或强制执行正式的应用程序概念相反，应用程序是非正式的，并使用元数据进行描述。应用程序包含的定义是松散的。一组通用的标签可以让多个工具之间相互操作，用所有工具都能理解的通用方式描述对象。

Label 是附着到 object 上（例如 Pod）的键值对。可以在创建 object 的时候指定，也可以在 object 创建后随时指定。Labels 的值对系统本身并没有什么含义，只是对用户才有意义。

```json
"labels": {
  "key1" : "value1",
  "key2" : "value2"
}
```

Kubernetes 最终将对 labels 最终索引和反向索引用来优化查询和 watch，在 UI 和命令行中会对它们排序。不要在 label 中使用大型、非标识的结构化数据，记录这样的数据应该用 annotation。Label 能够将组织架构映射到系统架构上（就像是康威定律），这样能够更便于微服务的管理，你可以给 object 打上如下类型的 label：

- "release" : "stable", "release" : "canary"
- "environment" : "dev", "environment" : "qa", "environment" : "production"
- "tier" : "frontend", "tier" : "backend", "tier" : "cache"
- "partition" : "customerA", "partition" : "customerB"
- "track" : "daily", "track" : "weekly"
- "team" : "teamA","team:" : "teamB"

# Label Selector

Label 不是唯一的，很多 object 可能有相同的 label。通过 label selector，客户端／用户可以指定一个 object 集合，通过 label selector 对 object 的集合进行操作。Label selector 有两种类型：

- equality-based：可以使用=、==、!=操作符，可以使用逗号分隔多个表达式
- set-based：可以使用 in、notin、!操作符，另外还可以没有操作符，直接写出某个 label 的 key，表示过滤有某个 key 的 object 而不管该 key 的 value 是何值，! 表示没有该 label 的 object

```sh
$ kubectl get pods -l environment=production,tier=frontend
$ kubectl get pods -l 'environment in (production),tier in (frontend)'
$ kubectl get pods -l 'environment in (production, qa)'
$ kubectl get pods -l 'environment,environment notin (frontend)'
```

## API Object

在 service、replicationcontroller 等 object 中有对 Pod 的 label selector，使用方法只能使用等于操作，例如：

```yml
selector:
  component: redis
```

在 Job、Deployment、ReplicaSet 和 DaemonSet 这些 object 中，支持 set-based 的过滤，例如：

```yml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - { key: tier, operator: In, values: [cache] }
    - { key: environment, operator: NotIn, values: [dev] }
```

如 Service 通过 label selector 将同一类型的 Pod 作为一个服务 expose 出来。

```yml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/e2e-az-name
              operator: In
              values:
                - e2e-az1
                - e2e-az2
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
            - key: another-node-label-key
              operator: In
              values:
                - another-node-label-value
```

# 推荐的标签

共享标签和注解都使用同一个前缀：app.kubernetes.io。没有前缀的标签是用户私有的。共享前缀可以确保共享标签不会干扰用户自定义的标签。为了充分利用这些标签，应该在每个资源对象上都使用它们：

| 键                             | 描述                                               | 示例               | 类型   |
| :----------------------------- | :------------------------------------------------- | :----------------- | :----- |
| `app.kubernetes.io/name`       | 应用程序的名称                                     | `mysql`            | 字符串 |
| `app.kubernetes.io/instance`   | 用于唯一确定应用实例的名称                         | `wordpress-abcxzy` | 字符串 |
| `app.kubernetes.io/version`    | 应用程序的当前版本（例如，语义版本，修订版哈希等） | `5.7.21`           | 字符串 |
| `app.kubernetes.io/component`  | 架构中的组件                                       | `database`         | 字符串 |
| `app.kubernetes.io/part-of`    | 此级别的更高级别应用程序的名称                     | `wordpress`        | 字符串 |
| `app.kubernetes.io/managed-by` | 用于管理应用程序的工具                             | `helm`             | 字符串 |

应用可以在 Kubernetes 集群中安装一次或多次。在某些情况下，可以安装在同一命名空间中。例如，可以不止一次地为不同的站点安装不同的 WordPress。应用的名称和实例的名称是分别记录的。例如，某 WordPress 实例的 app.kubernetes.io/name 为 wordpress，而其实例名称表现为 app.kubernetes.io/instance 的属性值 wordpress-abcxzy。这使应用程序和应用程序的实例成为可能是可识别的。应用程序的每个实例都必须具有唯一的名称。

## 案例：简单的无状态服务

考虑使用 Deployment 和 Service 对象部署的简单无状态服务的情况。以下两个代码段表示如何以最简单的形式使用标签。下面的 Deployment 用于监督运行应用本身的 pods。

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: myservice
    app.kubernetes.io/instance: myservice-abcxzy
```

下面的 Service 用于暴露应用。

```yml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: myservice
    app.kubernetes.io/instance: myservice-abcxzy
```

## 案例：带有一个数据库的 Web 应用程序

考虑一个稍微复杂的应用：一个使用 Helm 安装的 Web 应用（WordPress），其中 使用了数据库（MySQL）。以下代码片段说明用于部署此应用程序的对象的开始。以下 Deployment 的开头用于 WordPress：

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxzy
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
```

这个 Service 用于暴露 WordPress：

```yml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxzy
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
```

MySQL 作为一个 StatefulSet 暴露，包含它和它所属的较大应用程序的元数据：

```yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
```

`Service` 用于将 MySQL 作为 WordPress 的一部分暴露：

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
```

使用 MySQL `StatefulSet` 和 `Service`，您会注意到有关 MySQL 和 Wordpress 的信息，包括更广泛的应用程序。
