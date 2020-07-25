# Pod 执行与控制

# 容器创建与执行

我们可以创建如下的 Pod 配置文件：

```yml
apiVersion: v1
kind: Pod
metadata:
  name: shell-demo
spec:
  volumes:
    - name: shared-data
      emptyDir: {}
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html
  hostNetwork: true
  dnsPolicy: Default
```

注意，这里使用 hostNetwork 命令指明了使用所在主机的接口，我们接下来使用 kubectl 来创建 Pod：

```sh
$ kubectl apply -f https://k8s.io/examples/application/shell-demo.yaml

# 获取当前容器的状态
$ kubectl get pod shell-demo

# 执行 Pod 中的命令
$ kubectl exec -it shell-demo -- /bin/bash

# 单独执行某个指令
$ kubectl exec shell-demo env
$ kubectl exec shell-demo ps aux
$ kubectl exec shell-demo ls /
$ kubectl exec shell-demo cat /proc/1/mounts

# 当有多个容器时，可以指明执行某个容器的命令
$ kubectl exec -it my-pod --container main-app -- /bin/bash
```

这里我们使用了本地的 Volume 共享，我们可以在共享的目录中创建新的文本：

```sh
$ echo Hello shell demo > /usr/share/nginx/html/index.html
$ curl localhost
```

# 多个容器

在以下的配置中，我们可以创建包含两个容器的 Pod，这两个容器共享 Volume 并以此通信：

```yml
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:
  restartPolicy: Never

  volumes:
    - name: shared-data
      emptyDir: {}

  containers:
    - name: nginx-container
      image: nginx
      volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html

    - name: debian-container
      image: debian
      volumeMounts:
        - name: shared-data
          mountPath: /pod-data
      command: ["/bin/sh"]
      args:
        ["-c", "echo Hello from the debian container > /pod-data/index.html"]
```

在配置文件中，您可以看到 Pod 具有一个名为 shared-data 的卷。配置文件中列出的第一个容器运行 nginx 服务器。共享卷的安装路径为 `/usr/share/nginx/html`。第二个容器基于 debian 映像，并且具有 `/pod-data` 的安装路径。第二个容器运行以下命令，然后终止。

```sh
$ echo Hello from the debian container > /pod-data/index.html
```

```sh
$ kubectl get pod two-containers --output=yaml

apiVersion: v1
kind: Pod
metadata:
  ...
  name: two-containers
  namespace: default
  ...
spec:
  ...
  containerStatuses:

  - containerID: docker://c1d8abd1 ...
    image: debian
    ...
    lastState:
      terminated:
        ...
    name: debian-container
    ...

  - containerID: docker://96c1ff2c5bb ...
    image: nginx
    ...
    name: nginx-container
    ...
    state:
      running:
    ...
```

我们可以看到 debian container 被终止了，而 nginx container 依然运行。

```sh
$ kubectl exec -it two-containers -c nginx-container -- /bin/bash
$ root@two-containers:/# curl localhost

> Hello from the debian container
```
