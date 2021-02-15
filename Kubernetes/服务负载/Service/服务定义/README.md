# 服务注册与发现

Kubernetes ServiceTypes 允许指定一个需要的类型的 Service，默认是 ClusterIP 类型。Service 有三种类型：

- ClusterIP：默认类型，自动分配一个仅 Cluster 内部可以访问的虚拟 IP。
- NodePort：在 ClusterIP 基础上为 Service 在每台机器上绑定一个端口，这样就可以通过 `<NodeIP>:NodePort` 来访问该服务。
- LoadBalancer：在 NodePort 的基础上，借助 Cloud Provider 创建一个外部的负载均衡器，并将请求转发到 `<NodeIP>:NodePort`。
- ExternalName：通过返回 CNAME 和它的值，可以将服务映射到 externalName 字段的内容（例如，foo.bar.example.com）。没有任何类型代理被创建，这只有 Kubernetes 1.7 或更高版本的 kube-dns 才支持。

# Service 定义

一个 Service 在 Kubernetes 中是一个 REST 对象，和 Pod 类似。像所有的 REST 对象一样，Service 定义可以基于 POST 方式，请求 Api Server 创建新的实例。典型的 Service 定义方式如下：

```yml
kind: Service
apiVersion: v1
metadata:
  name: hostname-service
spec:
  # Expose the service on a static port on each node
  # so that we can access the service from outside the cluster
  type: NodePort

  # When the node receives a request on the static port (30163)
  # "select pods with the label 'app' set to 'echo-hostname'"
  # and forward the request to one of them
  selector:
    app: echo-hostname

  ports:
    # Three types of ports for a service
    # nodePort - a static port assigned on each the node
    # port - port exposed internally in the cluster
    # targetPort - the container port to send requests to
    - nodePort: 30163
      port: 8080
      targetPort: 80
```

这里需要对于 Port 的定义进行简要说明：

- port 表示 Service 暴露在 Cluster IP 上的端口，`<cluster ip>:port` 是提供给集群内部客户访问 Service 的入口。
- nodePort 是 K8s 提供给集群外部客户访问 Service 入口的一种方式（另一种方式是 LoadBalancer），所以，`<nodeIP>:nodePort` 是提供给集群外部客户访问 Service 的入口。
- targetPort 是 Pod 上的端口，从 port 和 nodePort 上到来的数据最终经过 kube-proxy 流入到后端 Pod 的 targetPort 上进入容器。

总的来说，port 和 nodePort 都是 Service 的端口，前者暴露给集群内客户访问服务，后者暴露给集群外客户访问服务。从这两个端口到来的数据都需要经过反向代理 kube-proxy 流入后端 Pod 的 targetPod，从而到达 Pod 上的容器内。默认情况下，targetPort 将被设置为与 port 字段相同的值；targetPort 可以是一个字符串，引用了后端 Pod 的一个端口的名称。但是，实际指派给该端口名称的端口号，在每个后端 Pod 中可能并不相同。对于部署和设计 Service，这种方式会提供更大的灵活性；例如，可以在后端 软件下一个版本中，修改 Pod 暴露的端口，并不会中断客户端的调用。

## 多端口 Service

很多 Service 需要暴露多个端口。对于这种情况，Kubernetes 支持在 Service 对象中定义多个端口。当使用多个端口时，必须给出所有端口的名称，这样 Endpoint 就不会产生歧义，例如：

```yml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

## 应用选择

Kubernetes Service 能够支持 TCP 和 UDP 协议，默认 TCP 协议，其通过标签来选取服务后端，一般配合 Replication Controller 或者 Deployment 来保证后端容器的正常运行。这一组 Pod 能够被 Service 访问到，通常是通过 Label Selector 实现的。另外，也可以将已有的服务以 Service 的形式加入到 Kubernetes 集群中来，只需要在创建 Service 的时候不指定 Label selector，而是在 Service 创建好后手动为其添加 endpoint。对 Kubernetes 集群中的应用，Kubernetes 提供了简单的 Endpoints API，只要 Service 中的一组 Pod 发生变更，应用程序就会被更新。对非 Kubernetes 集群中的应用，Kubernetes 提供了基于 VIP 的网桥的方式访问 Service，再由 Service 重定向到后端 Pod。

# NodePort 类型

如果设置 type 的值为 NodePort，Kubernetes master 将从给定的配置范围内（默认：30000-32767）分配端口，每个 Node 将从该端口（每个 Node 上的同一端口）代理到 Service。该端口将通过 Service 的 `spec.ports[*].nodePort` 字段被指定。如果需要指定的端口号，可以配置 nodePort 的值，系统将分配这个端口，否则调用 API 将会失败（比如，需要关心端口冲突的可能性）。这可以让开发人员自由地安装他们自己的负载均衡器，并配置 Kubernetes 不能完全支持的环境参数，或者直接暴露一个或多个 Node 的 IP 地址。需要注意的是，Service 将能够通过 `spec.ports[*].nodePort` 和 `spec.clusterIp:spec.ports[*].port` 而对外可见。

运行以下命令，使用 NodePort 服务导出 pod。

```sh
$ kubectl expose pods nginx --type=NodePort --port=80

service/nginx exposed

$ kubectl get svc

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        29m
nginx        NodePort    10.101.42.230   <none>        80:31858/TCP   5s

$ kubectl delete svc nginx

service "nginx" deleted
```

现在我们要创建一个 NodePort 服务，但我们要创建一个包含其定义的 yaml 清单。

```sh
vim primeiro-service-nodeport.yaml
```

```yaml
apiVersion : v1
kind : Service
metadata :
   labels :
     run : nginx
  name : nginx-nodeport
  namespace : default
spec :
   externalTrafficPolicy : Cluster
  ports :
  - nodePort : 31111
    port : 80
    protocol : TCP
    targetPort : 80
  selector :
     run : nginx
  sessionAffinity : None
  type : NodePort
```

然后创建服务：

```sh
$ kubectl create -f primeiro-service-nodeport.yaml

service/nginx-nodeport created

$ kubectl get services

NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP        30m
nginx-nodeport   NodePort    10.102.91.81   <none>        80:31111/TCP   7s

$ kubectl describe service nginx

Name:                     nginx-nodeport
Namespace:                default
Labels:                   run=nginx
Annotations:              <none>
Selector:                 run=nginx
Type:                     NodePort
IP:                       10.102.91.81
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31111/TCP
Endpoints:                10.46.0.1:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

$ kubectl delete -f primeiro-service-nodeport.yaml

service "nginx-nodeport" deleted
```

# LoadBalancer 类型

使用支持外部负载均衡器的云提供商的服务，设置 type 的值为 "LoadBalancer"，将为 Service 提供负载均衡器。负载均衡器是异步创建的，关于被提供的负载均衡器的信息将会通过 Service 的 status.loadBalancer 字段被发布出去。

```yml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
      nodePort: 30061
  clusterIP: 10.0.171.239
  loadBalancerIP: 78.11.24.19
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
      - ip: 146.148.47.155
```

来自外部负载均衡器的流量将直接打到后端 Pod 上，不过实际它们是如何工作的，这要依赖于云提供商。在这些情况下，将根据用户设置的 loadBalancerIP 来创建负载均衡器。某些云提供商允许设置 loadBalancerIP。如果没有设置 loadBalancerIP，将会给负载均衡器指派一个临时 IP。如果设置了 loadBalancerIP，但云提供商并不支持这种特性，那么设置的 loadBalancerIP 值将会被忽略掉。

# 其他类型的服务后端

Service 抽象了该如何访问 Kubernetes Pod，但也能够抽象其它类型的后端，例如：

- 希望在生产环境中使用外部的数据库集群，但测试环境使用自己的数据库。
- 希望服务指向另一个 Namespace 中或其它集群中的服务。
- 正在将工作负载转移到 Kubernetes 集群，和运行在 Kubernetes 集群之外的后端。

在任何这些场景中，都能够定义没有 selector 的 Service：

```yml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

由于这个 Service 没有 selector，就不会创建相关的 Endpoints 对象。可以手动将 Service 映射到指定的 Endpoints：

```yml
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 1.2.3.4
    ports:
      - port: 9376
```

注意，Endpoint IP 地址不能是 loopback（127.0.0.0/8）、link-local（169.254.0.0/16）、或者 link-local 多播（224.0.0.0/24）。访问没有 selector 的 Service，与有 selector 的 Service 的原理相同。请求将被路由到用户定义的 Endpoint（该示例中为 1.2.3.4:9376）。ExternalName Service 是 Service 的特例，它没有 selector，也没有定义任何的端口和 Endpoint。相反地，对于运行在集群外部的服务，它通过返回该外部服务的别名这种方式来提供服务。

```yml
kind: Service
apiVersion: v1
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

当查询主机 my-service.prod.svc.CLUSTER 时，集群的 DNS 服务将返回一个值为 my.database.example.com 的 CNAME 记录。访问这个服务的工作方式与其它的相同，唯一不同的是重定向发生在 DNS 层，而且不会进行代理或转发。如果后续决定要将数据库迁移到 Kubernetes 集群中，可以启动对应的 Pod，增加合适的 Selector 或 Endpoint，修改 Service 的 type。
