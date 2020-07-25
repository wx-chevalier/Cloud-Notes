# kubectl

kubeadm 是集群的安装配置脚手架，kubectl 是集群管理工具，kubelet 是工作节点上的代理 Daemon 服务, 负责与 Master 节点进行通信。kubectl 是我们日常工作中最常见的 Kubernetes 命令，本部分即对 kubectl 的常用操作进行总结。

## 自动补全

```sh
# Bash
source <(kubectl completion bash) # setup autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.

alias k=kubectl
complete -F __start_kubectl k

# ZSH
source <(kubectl completion zsh)  # setup autocomplete in zsh into the current shell
echo "if [ $commands[kubectl] ]; then source <(kubectl completion zsh); fi" >> ~/.zshrc # add autocomplete permanently to your zsh shell
```

## 插件

K8s 生态圈为我们提供了非常丰富的插件，这里我们可以使用 krew 作为 K8s 的插件安装工具：

```s
(
  set -x; cd "$(mktemp -d)" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/download/v0.3.3/krew.{tar.gz,yaml}" &&
  tar zxvf krew.tar.gz &&
  KREW=./krew-"$(uname | tr '[:upper:]' '[:lower:]')_amd64" &&
  "$KREW" install --manifest=krew.yaml --archive=krew.tar.gz &&
  "$KREW" update
)

export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

然后可以通过 krew 来安装插件：

```s
kubectl krew search                 # show all plugins
kubectl krew install view-secret    # install a plugin named "view-secret"
kubectl view-secret                 # use the plugin
kubectl krew upgrade                # upgrade installed plugins
kubectl krew uninstall view-secret  # uninstall a plugin
```

# 上下文配置

通过 kubectl 子命令 config 的三元组：集群（set-cluster）、用户（set-credentials）和配置上下文（set-context）实现切换。K8s 中的上下文能够连接用户与集群，如果通过 kubectl 的操作如下：

```sh
# 创建cluster
kubectl config set-cluster set-cluster scratch --server=https://5.6.7.8 --insecure-skip-tls-verify
# 创建user
kubectl config set-credentials  experimenter --username=exp --password=some-password
# 创建context
kubectl config set-context exp-scratch --cluster=scratch --namespace=default --user=experimenter
# 指定当前使用的context
kubectl config use-context exp-scratch
```

我们也可以指明如下的配置文件，通过 `export KUBECONFIG=/path/to/config.yml` 的方式来指明当前的上下文：

```yml
apiVersion: v1
kind: Config
preferences: {}

# Define the cluster
clusters:
  - cluster:
      certificate-authority-data: xx
      server: "https:/xx:6443"
    name: "xx"

# Define the user
users:
  - name: "xx"
    user:
      as-user-extra: {}
      client-key-data: "xx"
      token: "xx"

# Define the context: linking a user to a cluster
contexts:
  - context:
      cluster: "0"
      namespace: "xx"
      user: "xx"
    name: "xx"

# Define current context
current-context: "xx"
```

## 上下文切换

在 K8s 集群安装完毕之后，可以下载集群的配置文件到本地 kubectl 配置中：

```sh
mkdir $HOME/.kube
scp root@<master-public-ip>:/etc/kubernetes/kube.conf $HOME/.kube/config
```

然后可以来查看当前的上下文

```sh
$ unset KUBECONFIG
$ kubectl config current-context # 查看当前载入的上下文
$ kubectl config get-contexts # 浏览可用的上下文
$ kubectl config use-context context-name # 切换到指定上下文
```

在操作 Kubernetes 时，处理来自不同名称空间的资源，这也是一种很常见的做法。例如，你可能希望列出一个名称空间内的所有 Pod，随后检查另一个名称空间中的服务。此时我的做法是使用 Kubernetes CLI 所支持的 --namespace 标记。例如，若要查看名为 Test 的名称空间中的所有 Pod，可以运行 `kubectl get pods -n test`。默认情况下，如果不提供名称空间标记，将使用默认的 Kubernetes 名称空间，即 default。

这个默认值可以在 kubeconfig 文件中修改，例如我们可以将默认名称空间设置为 test、kube-system 或其他任何名称空间。这样在查询资源时就不需要使用 --namespace 标记了。不过更改默认值的命令略微繁琐：

```sh
$ kubectl config set contexts.my-context.namespace my-namespace
```

上述命令会更改 my-context 上下文的 Namespace 字段，将其改为 my-namespace。这也意味着，举例来说，如果切换到 my-context 随后运行 kubectl get pods，将只能看到 my-namespace 名称空间下的 Pod。除了使用 kubectx，我们还可以使用一款名为 kubens 的工具，后者可以帮助我们列出并切换至不同名称空间。

```sh
$ kubens
default
docker
kube-node-lease
kube-public
kube-system
```

为所选上下文设置默认名称空间，这也是一种快速简单的操作：

```sh
$ kubens default
Context "docker-desktop" modified.
Active namespace is "default".
```

# 运行与管理

`kubectl run` 和 docker run 一样，它能将一个镜像运行起来，我们使用 kubectl run 来将一个 sonarqube 的镜像启动起来。

```sh
$ kubectl run sonarqube --image=sonarqube:5.6.5 --replicas=1 --port=9000

deployment "sonarqube" created

# 该命令为我们创建了一个 Deployment
$ kubectl get deployment
NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
sonarqube   1         1         1            1           5m
```

我们也可以直接以交互方式运行某个镜像：

```sh
$ kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il
```

K8s 将镜像运行在 Pod 中以方便实施卷和网络共享等管理，使用 get pods 可以清楚的看到生成了一个 Pod：

```sh
$ kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
sonarqube-1880671902-s3fdq   1/1       Running   0          6m

# 交互式运行 Pod 中的某个命令
$ kubectl exec -it sonarqube-1880671902-s3fdq -- /bin/bash
```

`kubectl` 可以用于删除创建好的 Deployment 与 Pod：

```sh
$ kubectl delete pods sonarqube-1880671902-s3fdq
$ kubectl delete deployment sonarqube
```

最后，我们以创建包含两个 Pod 的应用为例，展示简单的 Pod 创建、暴露等操作:

```sh
# 创建 Deployment
$ kubectl run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080

# 获取 Deployment 相关的信息
$ kubectl get deployments hello-world
$ kubectl describe deployments hello-world

# 获取关联的 ReplicaSet 对象
$ kubectl get replicasets
$ kubectl describe replicasets

# 创建 Service 对象，并且暴露服务
$ kubectl expose deployment hello-world --type=NodePort --name=example-service

# 获取服务信息
$ kubectl describe services example-service

# 获取关联的 Pod 信息
$ kubectl get pods --selector="run=load-balancer-example" --output=wide

# 访问服务
$ curl http://<public-node-ip>:<node-port>
```

## 网络访问

```sh
# 暴露某个 Pod 的端口
$ kubectl port-forward -n NAMESPACE $POD <local-port>:<pod-port> &
$ kubectl port-forward $POD_NAME 8080:80

# 暴露某个 Service 的端口
$ kubectl -n rook-ceph port-forward service/rook-ceph-mgr-dashboard 31631:7000 --address 0.0.0.0
$ kubectl port-forward -n default deployment/postgres 8432:5432

$ export WEBAPP_POD=$(kubectl get pods -n $NAMESPACE | grep web-app | awk '{print $1;}')
$ kubectl port-forward -n $NAMESPACE $WEBAPP_POD 8080

$ kubectl proxy --port 8002
# http://localhost:8002/api/v1/proxy/namespaces/NAMESPACE/services/SERVICE_NAME:SERVICE_PORT/

$ curl -v -u username:password \
  --cacert ./ca.pem \
  --cert ./crt.pem \
  --key ./key.pem \
  https://api.CLUSTER_ID.k8s.gigantic.io/api/v1/namespaces/logging/services/elasticsearch:es/proxy/_stats
```

# 资源操作

## 对象创建

kubectl 可以基于 Yaml 文件进行应用的生命周期管理：

```sh
# 创建
$ kubectl create -f yamls/mysql.yaml
# 删除
$ kubectl delete -f yamls/mysql.yaml
# 同时创建多个
$ kubectl create -f yamls/
# 同时删除多个
$ kubectl delete -f yamls/

$ kubectl apply -f ./my-manifest.yaml            # create resource(s)
$ kubectl apply -f ./my1.yaml -f ./my2.yaml      # create from multiple files
$ kubectl apply -f ./dir                         # create resource(s) in all manifest files in dir
$ kubectl apply -f https://git.io/vPieo          # create resource(s) from url
$ kubectl create deployment nginx --image=nginx  # start a single instance of nginx
$ kubectl explain pods,svc                       # get the documentation for pod and svc manifests

# Create multiple YAML objects from stdin
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000000"
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep-less
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000"
EOF

# Create a secret with several keys
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: $(echo -n "s33msi4" | base64 -w0)
  username: $(echo -n "jane" | base64 -w0)
EOF
```

## 资源检索

get 命令用于获取集群的一个或一些 resource 信息。使用--help 查看详细信息。kubectl 的帮助信息、示例相当详细，而且简单易懂。建议大家习惯使用帮助信息。kubectl 可以列出集群所有 resource 的详细。resource 包括集群节点、运行的 pod，ReplicationController，service 等。

```sh
$ kubectl get [(-o|--output=)json|yaml|wide|go-template=...|go-template-file=...|jsonpath=...|jsonpath-file=...] (TYPE [NAME | -l label] | TYPE/NAME ...) [flags] [flags]
```

```sh
# Get commands with basic output
$ kubectl get services                          # List all services in the namespace
$ kubectl get pods --all-namespaces             # List all pods in all namespaces
$ kubectl get pods -o wide                      # List all pods in the namespace, with more details
$ kubectl get deployment my-dep                 # List a particular deployment
$ kubectl get pods                              # List all pods in the namespace
$ kubectl get pod my-pod -o yaml                # Get a pod's YAML
$ kubectl get pod my-pod -o yaml --export       # Get a pod's YAML without cluster specific information

# Describe commands with verbose output
$ kubectl describe nodes my-node
$ kubectl describe pods my-pod

# List Services Sorted by Name
$ kubectl get services --sort-by=.metadata.name

# List pods Sorted by Restart Count
$ kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# List pods in test namespace sorted by capacity
$ kubectl get pods -n test --sort-by=.spec.capacity.storage

# Get the version label of all pods with label app=cassandra
$ kubectl get pods --selector=app=cassandra -o \
  jsonpath='{.items[*].metadata.labels.version}'

# Get all worker nodes (use a selector to exclude results that have a label
# named 'node-role.kubernetes.io/master')
$ kubectl get node --selector='!node-role.kubernetes.io/master'

# Get all running pods in the namespace
$ kubectl get pods --field-selector=status.phase=Running

# Get ExternalIPs of all nodes
$ kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# List Names of Pods that belong to Particular RC
# "jq" command useful for transformations that are too complex for jsonpath, it can be found at https://stedolan.github.io/jq/
sel=${$(kubectl get rc my-rc --output=json | jq -j '.spec.selector | to_entries | .[] | "\(.key)=\(.value),"')%?}
echo $(kubectl get pods --selector=$sel --output=jsonpath={.items..metadata.name})

# Show labels for all pods (or any other Kubernetes object that supports labelling)
$ kubectl get pods --show-labels

# Check which nodes are ready
JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"

# List all Secrets currently in use by a pod
$ kubectl get pods -o json | jq '.items[].spec.containers[].env[]?.valueFrom.secretKeyRef.name' | grep -v null | sort | uniq

# List Events sorted by timestamp
$ kubectl get events --sort-by=.metadata.creationTimestamp
```

## 资源更新

```sh
$ kubectl set image deployment/frontend www=image:v2               # Rolling update "www" containers of "frontend" deployment, updating the image
$ kubectl rollout history deployment/frontend                      # Check the history of deployments including the revision
$ kubectl rollout undo deployment/frontend                         # Rollback to the previous deployment
$ kubectl rollout undo deployment/frontend --to-revision=2         # Rollback to a specific revision
$ kubectl rollout status -w deployment/frontend                    # Watch rolling update status of "frontend" deployment until completion

cat pod.json | kubectl replace -f -                              # Replace a pod based on the JSON passed into std

# Force replace, delete and then re-create the resource. Will cause a service outage.
$ kubectl replace --force -f ./pod.json

# Create a service for a replicated nginx, which serves on port 80 and connects to the containers on port 8000
$ kubectl expose rc nginx --port=80 --target-port=8000

# Update a single-container pod's image version (tag) to v4
$ kubectl get pod mypod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -

$ kubectl label pods my-pod new-label=awesome                      # Add a Label
$ kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq       # Add an annotation
$ kubectl autoscale deployment foo --min=2 --max=10

# Partially update a node
$ kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}'

# Update a container's image; spec.containers[*].name is required because it's a merge key
$ kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"new image"}]}}'

# Update a container's image using a json patch with positional arrays
$ kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path": "/spec/containers/0/image", "value":"new image"}]'

# Disable a deployment livenessProbe using a json patch with positional arrays
$ kubectl patch deployment valid-deployment  --type json   -p='[{"op": "remove", "path": "/spec/template/spec/containers/0/livenessProbe"}]'

# Add a new element to a positional array
$ kubectl patch sa default --type='json' -p='[{"op": "add", "path": "/secrets/1", "value": {"name": "whatever" } }]'
```

资源扩展：

```sh
$ kubectl scale --replicas=3 rs/foo                                 # Scale a replicaset named 'foo' to 3
$ kubectl scale --replicas=3 -f foo.yaml                            # Scale a resource specified in "foo.yaml" to 3
$ kubectl scale --current-replicas=2 --replicas=3 deployment/mysql  # If the deployment named mysql's current size is 2, scale mysql to 3
$ kubectl scale --replicas=5 rc/foo rc/bar rc/baz                   # Scale multiple replication controllers
```

## 资源删除

```sh
$ kubectl delete -f ./pod.json                                              # Delete a pod using the type and name specified in pod.json
$ kubectl delete pod,service baz foo                                        # Delete pods and services with same names "baz" and "foo"
$ kubectl delete pods,services -l name=myLabel                              # Delete pods and services with label name=myLabel
$ kubectl -n my-ns delete pod,svc --all                                      # Delete all pods and services in namespace my-ns,
# Delete all pods matching the awk pattern1 or pattern2
$ kubectl get pods  -n mynamespace --no-headers=true | awk '/pattern1|pattern2/{print $1}' | xargs  kubectl delete -n mynamespace pod
```

# 资源交互

## 与 Pod 交互

```sh
$ kubectl logs my-pod                                 # dump pod logs (stdout)
$ kubectl logs -l name=myLabel                        # dump pod logs, with label name=myLabel (stdout)
$ kubectl logs my-pod --previous                      # dump pod logs (stdout) for a previous instantiation of a container
$ kubectl logs my-pod -c my-container                 # dump pod container logs (stdout, multi-container case)
$ kubectl logs -l name=myLabel -c my-container        # dump pod logs, with label name=myLabel (stdout)
$ kubectl logs my-pod -c my-container --previous      # dump pod container logs (stdout, multi-container case) for a previous instantiation of a container
$ kubectl logs -f my-pod                              # stream pod logs (stdout)
$ kubectl logs -f my-pod -c my-container              # stream pod container logs (stdout, multi-container case)
$ kubectl logs -f -l name=myLabel --all-containers    # stream all pods logs with label name=myLabel (stdout)
$ kubectl run -i --tty busybox --image=busybox -- sh  # Run pod as interactive shell
$ kubectl run nginx --image=nginx --restart=Never -n
mynamespace                                         # Run pod nginx in a specific namespace
$ kubectl run nginx --image=nginx --restart=Never     # Run pod nginx and write its spec into a file called pod.yaml
--dry-run -o yaml > pod.yaml

$ kubectl attach my-pod -i                            # Attach to Running Container
$ kubectl port-forward my-pod 5000:6000               # Listen on port 5000 on the local machine and forward to port 6000 on my-pod
$ kubectl exec my-pod -- ls /                         # Run command in existing pod (1 container case)
$ kubectl exec my-pod -c my-container -- ls /         # Run command in existing pod (multi-container case)
$ kubectl top pod POD_NAME --containers               # Show metrics for a given pod and its containers
```

## 与 Node 及集群交互

```sh
kubectl cordon my-node                                                # Mark my-node as unschedulable
kubectl drain my-node                                                 # Drain my-node in preparation for maintenance
kubectl uncordon my-node                                              # Mark my-node as schedulable
kubectl top node my-node                                              # Show metrics for a given node
kubectl cluster-info                                                  # Display addresses of the master and services
kubectl cluster-info dump                                             # Dump current cluster state to stdout
kubectl cluster-info dump --output-directory=/path/to/cluster-state   # Dump current cluster state to /path/to/cluster-state

# If a taint with that key and effect already exists, its value is replaced as specified.
kubectl taint nodes foo dedicated=special-user:NoSchedule
```

# 其他操作

## 在 Kubernetes 集群中运行 terminal

在访问集群中的服务和 Pod 时，我们需要将其暴露出来，这样才可以从公网访问它们，或在本机和集群中运行的服务之间运行 Kube 代理或转发端口。然而有时候我们可能并不想暴露任何服务或转发端口，而只需要运行某些非常简单的 Curl 命令。为此我会通过 Bash profile 加载一个函数，借此在集群内部使用 radial/busyboxplus:curl 镜像运行一个 Pod，通过这样的方式就可以访问终端，进而可以针对集群内部的服务和 IP 运行 Curl 命令。我将这个函数称之为 kbash，用法如下：

```sh
$ kbash
If you don't see a command prompt, try pressing enter.
[ root@curl:/ ]$
```

在上述命令提示符下，我可以针对内部的 Kubernetes DNS 名称或 IP 地址运行 Curl 命令。如果需要退出，只需要运行 exit 即可；如果需要重新连接到该 Pod，则可运行 kbash 连接到现有 Pod。同时我还将这个函数定义到了自己的 dotfiles 中。

## 快速打开 Grafana/Jaeger/Kiali

如果打算使用 Istio 服务网格（Service mesh），那么可能还会用到 Grafana/Jaeger/Kiali。访问这些服务时必需首先获得 Pod 名称，随后针对该 Pod 设置端口转发，最后才能打开浏览器访问转发后的地址。每次需要输入的命令都很长：

```sh
$ kubectl get pods --namespace istio-system -l "app=grafana" -o jsonpath="{.items[0].metadata.name}"
grafana-6fb9f8c5c7-hrcqp
$ kubectl --namespace istio-system port-forward grafana-6fb9f8c5c7-hrcqp 3000:3000
$ open http://localhost:3000
```

而更简单快捷的方法是为每个服务创建函数或别名。例如，我通过使用 Bash profile 加载的一个文件为 Grafana/Jaeger/Kiali 添加了如下设置：

```sh
#!/bin/bash
export GRAFANA_POD=$(kubectl get pods --namespace istio-system -l "app=grafana" -o jsonpath="{.items[0].metadata.name}")
export JAEGER_POD=$(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}')
export KIALI_POD=$(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}')
alias grafana="kubectl --namespace istio-system port-forward $GRAFANA_POD 3000:3000 & open http://localhost:3000"
alias jaeger="kubectl --namespace istio-system port-forward $JAEGER_POD 16686:16686 & open http://localhost:16686"
alias kiali="kubectl --namespace istio-system port-forward $KIALI_POD 20001:20001 & open http://localhost:20001"
```

这样，如果需要打开 Jaeger，只需要运行 jaeger 就可以获得 Pod 名称，创建端口转发并打开浏览器。如果你在集群中运行了其他什么需要频繁访问的服务，也可以用类似方式来设置别名。

# 链接

- https://blog.csdn.net/xingwangc2014/article/details/51204224
