# ReplicaSet

ReplicaSet 可以保证 Deployment 所需的 pod 和资源数量。一旦创建了部署，ReplicaSet 就会控制运行的 pod 数量，如果有任何 pod 完成，它将检测并请求执行另一个 pod，从而保证请求的复制数量。让我们创建第一个 ReplicaSet：

```yaml
vim primeiro-replicaset.yaml

apiVersion : apps / v1
kind : ReplicaSet
metadata :
   name : replica-set-first
spec :
   replicas : 3
  selector :
     matchLabels :
       system : Giropops
  template :
     metadata :
       labels :
         system : Giropops
    spec :
       containers :
      - name : nginx
        image : nginx: 1.7.9
        ports :
        - containerPort : 80
```

从 manifest 中创建 ReplicaSet：

```sh
$ kubectl create -f primeiro-replicaset.yaml

replicaset.extensions/replica-set-primeiro created

$ kubectl get replicaset

NAME                   DESIRED   CURRENT   READY    AGE
replica-set-primeiro   3         3         1        2s

$ kubectl get pods

NAME                         READY     STATUS    RESTARTS   AGE
replica-set-primeiro-6drmt   1/1       Running   0          12s
replica-set-primeiro-7j59w   1/1       Running   0          12s
replica-set-primeiro-mg8q9   1/1       Running   0          12s

$ kubectl describe rs replica-set-primeiro

Name:         replica-set-primeiro
Namespace:    default
Selector:     system=Giropops
Labels:       system=Giropops
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  system=Giropops
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  31s   replicaset-controller  Created pod: replica-set-primeiro-mg8q9
  Normal  SuccessfulCreate  31s   replicaset-controller  Created pod: replica-set-primeiro-6drmt
  Normal  SuccessfulCreate  31s   replicaset-controller  Created pod: replica-set-primeiro-7j59w
```

因此，我们可以看到所有与 ReplicaSet 相关联的 Pods，如果我们删除其中一个 Pods，会发生什么？让我们测试一下。

```sh
$ kubectl delete pod replica-set-primeiro-6drmt

pod "replica-set-primeiro-6drmt" deleted

$ kubectl get pods -l system=Giropops

NAME                         READY     STATUS    RESTARTS   AGE
replica-set-primeiro-7j59w   1/1       Running   0          1m
replica-set-primeiro-mg8q9   1/1       Running   0          1m
replica-set-primeiro-s5dz2   1/1       Running   0          15s
```

你有没有注意到他又重新制作了一个 Pod？ReplicaSet 的原因总是有三个 Pod 可用。我们将改为 4 个副本并重新创建 ReplicaSet，为此我们将使用之前的 kubectl edit，这样我们就可以更改已经运行的 ReplicaSet。

```sh
$ kubectl edit rs replica-set-primeiro

apiVersion : apps / v1
kind : ReplicaSet
metadata :
   creationTimestamp : 2018-07-05T04: 32: 42Z
  generation : 2
  labels :
     system : Giropops
  name : replica-set-first
  namespace : default
  resourceVersion : " 471758 "
   selfLink : / apis / extensions / v1beta1 / namespaces / default / replicasets / replica-set-first
  uid : 753290c1-800c-11e8-b889-42010a8a0002
spec :
   replicas: 4
  selector :
     matchLabels :
       system : Giropops
  template :
     metadata :
       creationTimestamp : null
      labels :
         system : Giropops
...

replicaset.extensions / replica-set-first edited
```

查看 Pod 详情。

```sh
$ kubectl get pods -l system=Giropops

NAME                         READY     STATUS    RESTARTS   AGE
replica-set-primeiro-7j59w   1/1       Running   0          2m
replica-set-primeiro-96hj7   1/1       Running   0          10s
replica-set-primeiro-mg8q9   1/1       Running   0          2m
replica-set-primeiro-s5dz2   1/1       Running   0          1m
```
