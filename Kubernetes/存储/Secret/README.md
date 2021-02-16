# Secret

Secret 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 ssh key。将这些信息放在 secret 中比放在 Pod 的定义或者 容器镜像 中来说更加安全和灵活。这样的信息可能会被放在 Pod spec 中或者镜像中；将其放在一个 secret 对象中可以更好地控制它的用途，并降低意外暴露的风险。用户可以创建 secret，同时系统也创建了一些 secret。要使用 secret，pod 需要引用 secret。

# 简单使用

Pod 可以用两种方式使用 secret：作为 volume 中的文件被挂载到 pod 中的一个或者多个容器里，或者当 kubelet 为 pod 拉取镜像时使用。

## 使用 Volume

```sh
$ echo -n "giropops strigus girus" > secret.txt
$ kubectl create secret generic my-secret --from-file=secret.txt

secret/my-secret created
```

我们来看看那个物体的细节，看看到底发生了什么。

```sh
$ kubectl describe secret my-secret

Name:         my-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
secret.txt:  18 bytes
```

需要注意的是，不能使用 describe 查看文件的内容，这是为了保护密钥不被意外暴露。要检查一个 Secret 的内容，我们需要对生成的文件进行解码，要做到这一点，我们必须检查同一文件的清单。

```sh
$ kubectl get secret

NAME              TYPE             DATA      AGE
my-secret         Opaque           1         13m

$ kubectl get secret my-secret -o yaml

apiVersion: v1
data:
  secret.txt: Z2lyb3BvcHMgc3RyaWd1cyBnaXJ1cw==
kind: Secret
metadata:
  creationTimestamp: 2018-08-26T17:10:14Z
  name: my-secret
  namespace: default
  resourceVersion: "3296864"
  selfLink: /api/v1/namespaces/default/secrets/my-secret
  uid: e61d124a-a952-11e8-8723-42010a8a0002
type: Opaque
```

现在我们有了加密后的密钥，只需使用 Base64 解密即可。

```sh
$ echo 'Z2lyb3BvcHMgc3RyaWd1cyBnaXJ1cw==' | base64 --decode

giropops strigus girus
```

好了，有了我们的 Secret，现在我们将在 Pod 里面使用它，为此我们需要在 Pod 里面使用卷来引用 Secret，我们将创建我们的清单。

```sh
vim pod-secret.yaml
```

```yaml
apiVersion : v1
kind : Pod
metadata :
   name : test-secret
  namespace : default
spec :
   containers :
  - image: busybox
    name: busy
    command:
      - sleep
      - "3600"
     volumeMounts :
        - mountPath: /tmp/giropops
        name : my-volume-secret
  volumes :
    - name : my-volume-secret
        secret :
        secretName : my-secret
```

在此清单中，我们将使用卷 my-volume-secret 将 Secret 挂载在容器 my-secret 目录/tmp/giropos 内。

```sh
$ kubectl create -f pod-secret.yaml

pod/test-secret created

$ kubectl exec -ti test-secret -- ls /tmp/giropops

secret.txt

$ kubectl exec -ti test-secret -- cat /tmp/giropops/secret.txt

giropops strigus girus
```

成功了！这就是我们在 Pods 中放置信息或密码的方法之一。

## 使用环境变量

这是把信息或密码放在我们的 Pod 里面的方法之一，但是还有一个更酷的方法，就是把 Secret 作为一个环境变量。让我们来看看这个家伙，首先让我们用字面键创建一个新的对象 Secret，有键和值。

```sh
$ kubectl create secret generic my-literal-secret --from-literal user=linuxtips --from-literal password=catota

secret/my-literal-secret created

$ kubectl describe secret my-literal-secret

Name:         my-literal-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  6 bytes
user:      9 bytes
```

我们刚刚创建了一个有两个密钥的对象 Secret，一个是用户，另一个是密码，现在我们将使用环境变量在 Pod 中引用这个密钥，为此我们将创建新的清单。

```sh
vim pod-secret-env.yaml
```

```yaml
apiVersion : v1
kind : Pod
metadata :
   name : test-secret-env
  namespace : default
spec :
   containers :
  - image : busybox
    name : busy-secret-env
    command :
      - sleep
      - " 3600 "
     env :
        - name : MEU_USERNAME
          valueFrom :
            secretKeyRef :
                name : my-literal-secret
                key : user
        - name : MEU_PASSWORD
          valueFrom :
            secretKeyRef :
                name : my-literal-secret
                key : password
```

```sh
$ kubectl create -f pod-secret-env.yaml

pod/teste-secret-env created

$ kubectl exec teste-secret-env -c busy-secret-env -it -- printenv

PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=teste-secret-env
TERM=xterm
MEU_USERNAME=linuxtips
MEU_PASSWORD=catota
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
HOME=/root
```

看到了吗？现在我们可以把这个 key 在容器里面作为一个环境变量，如果容器里面的任何一个应用需要连接到数据库，比如说使用用户名和密码，只要用这个信息创建一个 secret，并在 Pod 中引用，然后在 Pod 里面作为一个环境变量或者一个文本文件创建卷来消耗就可以了。

# TBD

- https://kubernetes.io/zh/docs/concepts/configuration/secret/
