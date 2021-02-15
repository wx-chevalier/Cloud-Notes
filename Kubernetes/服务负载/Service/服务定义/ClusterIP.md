# ClusterIP

我们将使用以下命令从一个 pod 模板创建一个 pod。

```sh
$ kubectl run nginx --image nginx --dry-run=client -o yaml > pod-template.yaml
$ kubectl create -f pod-template.yaml

pod/nginx created

$ kubectl expose pod nginx --port=80

service/nginx exposed

$ kubectl get svc

NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   25m
nginx        ClusterIP   10.104.209.243   <none>        80/TCP    7m15s
```

运行以下命令查看 nginx 服务的详细信息。

```sh
$ kubectl describe service nginx

Name:              nginx
Namespace:         default
Labels:            run=nginx
Annotations:       <none>
Selector:          run=nginx
Type:              ClusterIP
IP:                10.104.209.243
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.46.0.0:80
Session Affinity:  None
Events:            <none>
```

访问 Ningx。根据自己的环境，用下面的命令更改集群 IP。

```sh
$ curl 10.104.209.243

...
<title>Welcome to nginx!</title>
...

$ kubectl logs -f nginx

10.40.0.0 - - [10/May/2020:17:31:56 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.58.0" "-"

$ kubectl delete svc nginx

service "nginx" deleted
```

然后使用 `vim primeiro-service-clusterip.yaml`：

```yml
apiVersion : v1
kind : Service
metadata :
   labels :
     run : nginx
  name : nginx-clusterip
  namespace : default
spec :
   ports :
  - port : 80
    protocol : TCP
    targetPort : 80
  selector :
     run : nginx
  type : ClusterIP
```

创建服务：

```sh
$ kubectl create -f primeiro-service-clusterip.yaml

service/nginx-clusterip created

$ kubectl get services

NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP   28m
nginx-clusterip   ClusterIP   10.109.70.243   <none>        80/TCP    71s

$ kubectl describe service nginx-clusterip

Name:              nginx-clusterip
Namespace:         default
Labels:            run=nginx
Annotations:       <none>
Selector:          run=nginx
Type:              ClusterIP
IP:                10.109.70.243
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.46.0.1:80
Session Affinity:  None
Events:            <none>

$ kubectl delete -f primeiro-service-clusterip.yaml

service "nginx-clusterip" deleted
```

然后我们修改下 sessionAffinity 属性：

```yml
apiVersion : v1
kind : Service
metadata :
   labels :
     run : nginx
  name : nginx-clusterip
  namespace : default
spec :
   ports :
  - port : 80
    protocol : TCP
    targetPort : 80
  selector :
     run : nginx
  sessionAffinity : ClientIP
  type : ClusterIP
```

再次创建服务：

```sh
$ kubectl create -f primeiro-service-clusterip.yaml

service/nginx-clusterip created

$ kubectl get services

NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes        ClusterIP   10.96.0.1      <none>        443/TCP   29m
nginx-clusterip   ClusterIP   10.96.44.114   <none>        80/TCP    7s

$ kubectl describe service nginx

Name:              nginx-clusterip
Namespace:         default
Labels:            run=nginx
Annotations:       <none>
Selector:          run=nginx
Type:              ClusterIP
IP:                10.96.44.114
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.46.0.1:80
Session Affinity:  ClientIP
Events:            <none>
```

有了这些，现在我们就可以维护会话了，也就是说，它将与同一个 pod 保持连接，尊重客户端的原 IP。如果有必要，可以将超时值改为 sessionAffinity(默认值为 10800 秒，即 3 小时)，只需添加以下配置即可。

```yaml
sessionAffinityConfig:
  clientIP:
    timeoutSeconds: 10
```

现在我们可以删除服务。

```sh
$ kubectl delete -f primeiro-service-clusterip.yaml

service "nginx-clusterip" deleted
```

# EndPoint

每当我们创建一个服务，就会自动创建一个端点。端点无非就是服务要使用的 IP pod，比如我们创建服务类型 ClusterIP 的时候就有你的 IP，对吧？现在，当我们打到这个 IP 的时候，它就会通过这个 IP，即 EndPoint 重定向连接到 Pod。要列出已创建的 EndPoints，请运行命令。

```sh
$ kubectl get endpoints

NAME         ENDPOINTS         AGE
kubernetes   10.142.0.5:6443   4d

$ kubectl describe endpoints kubernetes

Name:         kubernetes
Namespace:    default
Labels:       <none>
Annotations:  <none>
Subsets:
  Addresses:          172.31.17.67
  NotReadyAddresses:  <none>
  Ports:
    Name   Port  Protocol
    ----   ----  --------
    https  6443  TCP

Events:  <none>
```

让我们做一个例子，对于这个，我们将执行一个部署的创建，将副本的数量增加到 3 个，然后是一个服务，这样我们就可以更详细地看到将创建的端点。

```sh
$ kubectl create deployment nginx --image=nginx

deployment.apps/nginx created

$ kubectl get deployments.apps

NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           5s
```

将 nginx 部署扩展到 3 个副本。

```sh
$ kubectl scale deployment nginx --replicas=3

deployment.apps/nginx scaled

$ kubectl get deployments.apps

NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           1m5s

$ kubectl expose deployment nginx --port=80

service/nginx exposed

$ kubectl get svc

NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   40m
nginx        ClusterIP   10.98.153.22   <none>        80/TCP    6s
```

访问 nginx：

```sh
curl 10.98.153.22

...
<h1>Welcome to nginx!</h1>
...
```

查看端点：

```sh
kubectl get endpoints

NAME         ENDPOINTS                                AGE
kubernetes   172.31.17.67:6443                        44m
nginx        10.32.0.2:80,10.32.0.3:80,10.46.0.2:80   3m31s
```

查看 nginx 端点的详细信息：

```sh
$ kubectl describe endpoints nginx

Name:         nginx
Namespace:    default
Labels:       app=nginx
Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2020-05-10T17:47:05Z
Subsets:
  Addresses:          10.32.0.2,10.32.0.3,10.46.0.2
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  80    TCP

Events:  <none>
```

以 YAML 格式查看端点。

```yaml
$ kubectl get endpoints -o yaml

apiVersion: v1
items:
- apiVersion: v1
  kind: Endpoints
  metadata:
    creationTimestamp: "2020-05-10T17:06:12Z"
    managedFields:
    - apiVersion: v1
      fieldsType: FieldsV1
      fieldsV1:
        f:subsets: {}
      manager: kube-apiserver
      operation: Update
      time: "2020-05-10T17:06:12Z"
    name: kubernetes
    namespace: default
    resourceVersion: "163"
    selfLink: /api/v1/namespaces/default/endpoints/kubernetes
    uid: 39f1e237-f9cc-4553-a32d-95402ff52f6c
...
    - ip: 10.46.0.2
      nodeName: elliot-03
      targetRef:
        kind: Pod
        name: nginx-f89759699-dmt4t
        namespace: default
        resourceVersion: "6805"
        uid: 6a9c4639-78ee-44c6-8eb1-4fd90d308189
    ports:
    - port: 80
      protocol: TCP
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

移除相关的资源：

```sh
$ kubectl delete deployment nginx

deployment.apps "nginx" deleted

$ kubectl delete service nginx

service "nginx" deleted
```
