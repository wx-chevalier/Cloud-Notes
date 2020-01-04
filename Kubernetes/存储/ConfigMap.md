# ConfigMap

应用部署的一个最佳实践是将应用所需的配置信息与程序进行分离，这样可以使得应用程序被更好地复用，通过不同的配置也能实现更灵活的功能。将应用打包为容器镜像后，可以通过环境变量或者外挂文件的方式在创建容器时进行配置注入，但在大规模容器集群的环境中，对多个容器进行不同的配置将变得非常复杂。ConfigMap 即是用来存储配置文件的 kubernetes 资源对象，所有的配置内容都存储在 etcd 中。

![ConfigMap 示意图](https://matthewpalmer.net/kubernetes-app-developer/articles/configmap-diagram.gif)

ConfigMap 供容器使用的典型用法如下：

- 生成为容器内的环境变量；
- 设置容器启动命令的启动参数（需设置为环境变量）；
- 以 Volume 的形式挂载为容器内部的文件或目录。

# 创建 ConfigMap

## 通过 yaml 配置文件方式创建`

```yml
kind: ConfigMap
apiVersion: v1
metadata:
  name: example-configmap
data:
  # Configuration values can be set as key-value properties
  database: mongodb
  database_uri: mongodb://localhost:27017

  # Or set as complete file contents (even JSON!)
  keys: |
    image.public.key=771 
    rsa.public.key=42
```

`kubectl apply -f config-map.yaml`

## 通过 kubectl 命令行方式创建

# 使用 ConfigMap

## 挂载为卷使用

```yml
kind: Pod
apiVersion: v1
metadata:
  name: pod-using-configmap

spec:
  # Add the ConfigMap as a volume to the Pod
  volumes:
    # `name` here must match the name
    # specified in the volume mount
    - name: example-configmap-volume
      # Populate the volume with config map data
      configMap:
        # `name` here must match the name
        # specified in the ConfigMap's YAML
        name: example-configmap

  containers:
    - name: container-configmap
      image: nginx:1.7.9
      # Mount the volume that contains the configuration data
      # into your container filesystem
      volumeMounts:
        # `name` here must match the name
        # from the volumes section of this pod
        - name: example-configmap-volume
            mountPath: /etc/config
```

## 作为环境变量使用

```yml
kind: Pod
apiVersion: v1
metadata:
  name: pod-env-var
spec:
  containers:
    - name: env-var-configmap
      image: nginx:1.7.9
      envFrom:
        - configMapRef:
            name: example-configmap
```

# 案例

## 挂载配置文件

我们也可以将文件创建为 ConfigMap，然后将其挂载到容器对应的卷中：

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sherlock-config
  namespace: default
data:
  config.yaml: |
    namespaces:
      - default
    labels:
      - "app"
      - "owner"
```

然后在 Pod 中创建关联的卷，指明文件路径：

```yml
apiVersion: v1
kind: Pod
metadata:
  name: kube-sherlock
spec:
  serviceAccountName: kube-sherlock
  containers:
    - name: kube-sherlock
      image: cmendibl3/kube-sherlock:0.1
      volumeMounts:
        - name: config-volume
          mountPath: /app/config.yaml
          subPath: config.yaml
  volumes:
    - name: config-volume
      configMap:
        name: sherlock-config
  restartPolicy: Never
```

# 链接

- https://learning.oreilly.com/library/view/kubernetes-for-developers/9781788834759/961e9251-74a4-4e47-8a48-9c02f3500bbb.xhtml
- https://blog.csdn.net/liukuan73/article/details/79492374
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
- https://www.jianshu.com/p/d834bca35c18
