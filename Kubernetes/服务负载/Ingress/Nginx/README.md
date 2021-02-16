# Ingress 配置

# 基于 K8s 内置的 Nginx Ingress 部署

Your service may have its own chart, or you may be deploying it directly with manifests. This quickstart uses manifests to create and expose a sample service. The example service uses [kuard](https://github.com/kubernetes-up-and-running/kuard), a demo application which makes an excellent back-end for examples.

The quickstart example uses three manifests for the sample. The first two are a sample deployment and an associated service:

- deployment manifest: [deployment.yaml](https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/docs/tutorials/acme/quick-start/example/deployment.yaml)

```yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: kuard
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: kuard
    spec:
      containers:
        - image: gcr.io/kuar-demo/kuard-amd64:1
          imagePullPolicy: Always
          name: kuard
          ports:
            - containerPort: 8080
```

- service manifest: [service.yaml](https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/docs/tutorials/acme/quick-start/example/service.yaml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kuard
spec:
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  selector:
    app: kuard
```

You can create download and reference these files locally, or you can reference them from the GitHub source repository for this documentation. To install the example service from the tutorial files straight from GitHub, you may use the commands:

```
$ kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/docs/tutorials/acme/quick-start/example/deployment.yaml
deployment.extensions "kuard" created

$ kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/docs/tutorials/acme/quick-start/example/service.yaml
service "kuard" created
```

An [ingress resource](https://kubernetes.io/docs/concepts/services-networking/ingress/) is what Kubernetes uses to expose this example service outside the cluster. You will need to download and modify the example manifest to reflect the domain that you own or control to complete this example.

A sample ingress you can start with is:

- ingress manifest: [ingress.yaml](https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/docs/tutorials/acme/quick-start/example/ingress.yaml)

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kuard
  annotations:
    kubernetes.io/ingress.class: "nginx"
    #certmanager.k8s.io/issuer: "letsencrypt-staging"

spec:
  tls:
    - hosts:
        - example.example.com
      secretName: quickstart-example-tls
  rules:
    - host: example.example.com
      http:
        paths:
          - path: /
            backend:
              serviceName: kuard
              servicePort: 80
```

You can download the sample manifest from github, edit it, and submit the manifest to Kubernetes with the command:

```
$ kubectl create --edit -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/docs/tutorials/acme/quick-start/example/ingress.yaml

# edit the file in your editor, and once it is saved:
ingress.extensions "kuard" created
```

Note

The ingress example we show above has a host definition within it. The nginx-ingress-controller will route traffic when the hostname requested matches the definition in the ingress. You *can*deploy an ingress without a host definition in the rule, but that pattern isn’t usable with a TLS certificate, which expects a fully qualified domain name.

- Once it is deployed, you can use the command kubectl get ingress to see the status

  of the ingress:

```
NAME      HOSTS     ADDRESS   PORTS     AGE
kuard     *                   80, 443   17s
```

It may take a few minutes, depending on your service provider, for the ingress to be fully created. When it has been created and linked into place, the ingress will show an address as well:

```
NAME      HOSTS     ADDRESS         PORTS     AGE
kuard     *         35.199.170.62   80        9m
```

Note

The IP address on the ingress _may not_ match the IP address that the nginx-ingress-controller. This is fine, and is a quirk/implementation detail of the service provider hosting your Kubernetes cluster. Since we are using the nginx-ingress-controller instead of any cloud-provider specific ingress backend, use the IP address that was defined and allocated for the nginx-ingress-service LoadBalancer resource as the primary access point for your service.

Make sure the service is reachable at the domain name you added above, for example http://example.your-domain.com. The simplest way is to open a browser and enter the name that you set up in DNS, and for which we just added the ingress.

You may also use a command line tool like curl to check the ingress.

```
$ curl -kivL -H 'Host: example.your-domain.com' 'http://35.199.164.14'
```

The options on this curl command will provide verbose output, following any redirects, show the TLS headers in the output, and not error on insecure certificates. With nginx-ingress-controller, the service will be available with a TLS certificate, but it will be using a self-signed certificate provided as a default from the nginx-ingress-controller. Browsers will show a warning that this is an invalid certificate. This is expected and normal, as we have not yet used cert-manager to get a fully trusted certificate for our site.

Warning

It is critical to make sure that your ingress is available and responding correctly on the internet. This quickstart example uses Let’s Encypt to provide the certificates, which expects and validates both that the service is available and that during the process of issuing a certificate uses that valdiation as proof that the request for the domain belongs to someone with sufficient control over the domain.

# 手动部署 Nginx Ingress

通常我们在 Kubernetes 中运行 Pod 时，所有的流量都只通过集群网络进行路由，所有的外部流量最终都会被丢弃或转发到其他位置。入口是一组规则，用于允许传入的外部连接到达集群内的服务。

## 定义服务

我们将创建第一个 Ingress，但首先我们将生成两个部署和两个服务。

```yaml
# vim app1.yaml
apiVersion : apps / v1
kind : Deployment
metadata :
    name : app1
spec :
    replicas : 2
  selector :
      matchLabels :
        app : app1
  template :
      metadata :
        labels :
          app : app1
    spec :
        containers :
      - image : dockersamples/static-site
        name : app1
        env :
        - name : AUTHOR
          value : GIROPOPS
        ports :
        - containerPort : 80
```

```yaml
# vim app2.yaml
apiVersion : apps / v1
kind : Deployment
metadata :
    name : app2
spec :
    replicas : 2
  selector :
      matchLabels :
        app : app2
  template :
      metadata :
        labels :
          app : app2
    spec :
        containers :
      - image : dockersamples/static-site
        name : app2
        env :
        - name : AUTHOR
          value : STRIGUS
        ports :
        - containerPort : 80
```

我们将使用以下命令在集群中创建部署：

```sh
$ kubectl create -f app1.yaml

deployment.apps/app1 created

$ kubectl create -f app2.yaml

deployment.apps/app2 created
```

然后配置服务：

```yaml
# vim svc-app1.yaml

apiVersion : v1
kind : Service
metadata :
    name : appsvc1
spec :
    ports :
  - port : 80
    protocol : TCP
    targetPort : 80
  selector :
      app : app1
```

```yaml
# vim svc-app2.yaml
apiVersion : v1
kind : Service
metadata :
    name : appsvc2
spec :
    ports :
  - port : 80
    protocol : TCP
    targetPort : 80
  selector :
      app : app2
```

让我们用以下命令在集群中创建服务：

```sh
$ kubectl create -f svc-app1.yaml

service/appsvc1 created

$ kubectl create -f svc-app2.yaml

service/appsvc2 created
```

我们刚刚从一个静态网站上创建了两个 Pod。

```sh
$ kubectl get deploy

NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
app1      2         2         2            2           3m
app2      2         2         2            2           3m

$ kubectl get services

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
appsvc1      ClusterIP   10.107.228.40   <none>        80/TCP    2m
appsvc2      ClusterIP   10.97.250.131   <none>        80/TCP    2m
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   11d
```

让我们列出服务的 Endpoints：

```sh
$ kubectl get ep

NAME         ENDPOINTS                     AGE
appsvc1      10.44.0.11:80,10.44.0.12:80   4m
appsvc2      10.32.0.4:80,10.44.0.13:80    4m
kubernetes   10.142.0.5:6443               11d
```

现在让我们访问这些站点，看看我们在 Deployments 中设置的环境变量是否一切顺利。

```sh
$ curl 10.44.0.11

...
<h1 id="toc_0">Hello GIROPOPS!</h1>

<p>This is being served from a <b>docker</b><br>
container running Nginx.</p>

$ curl  10.32.0.4

h1 id="toc_0">Hello STRIGUS!</h1>

<p>This is being served from a <b>docker</b><br>
container running Nginx.</p>
```

## 定义后端

让我们为后台创建一个部署：

```sh
$ vim default-backend.yaml
```

```yaml
apiVersion : apps/v1
kind : Deployment
metadata :
    name : default-backend
spec :
    replicas : 2
  selector :
      matchLabels :
        app : default-backend
  template :
      metadata :
        labels :
          app : default-backend
    spec :
        terminationGracePeriodSeconds : 60
      containers :
      - name : default-backend
        image : gcr.io/google_containers/defaultbackend:1.0
        livenessProbe :
            httpGet :
              path : /healthz
            port : 8080
            scheme : HTTP
          initialDelaySeconds : 30
          timeoutSeconds : 5
        ports :
        - containerPort : 8080
        resources :
            limits :
              cpu : 10m
            memory : 20Mi
          requests :
              cpu : 10m
            memory : 20Mi
```

注意前面文件中的以下参数。

- terminationGracePeriodSeconds => 在用 SIGTERM 信号执行强制终止之前，它将等待 pod 完成的时间，以秒为单位。
- livenessProbe => 检查 pod 是否还在运行，如果不在运行，它 kubelet 将移除容器并启动另一个容器。
- readnessProbe => 检查容器是否准备好接收服务的请求。
- initialDelaySeconds => 告诉 kubele 应该等待多少秒来执行第一次 livenessProbe 检查。
- timeoutSeconds => 被认为是探针执行超时的时间（以秒为单位），默认值为 1。
- periodSeconds => 确定检查 livenessProbe 的频率。

## 定义 Nginx Ingress

然后创建 ingress 相关：

```sh
$ kubectl create namespace ingress

namespace/ingress created

$ kubectl create -f default-backend.yaml -n ingress

deployment.apps/default-backend created

$ vim default-backend-service.yaml
```

```yaml
apiVersion : v1
kind : Service
metadata :
    name : default-backend
spec :
    ports :
  - port : 80
    protocol : TCP
    targetPort : 8080
  selector :
      app : default-backend
```

在命名空间 ingress 中为后台创建服务。

```sh
$ kubectl create -f default-backend-service.yaml -n ingress

service/default-backend created

$ kubectl get deployments.

NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
app1      2         2         2            2           29m
app2      2         2         2            2           28m

$ kubectl get deployments. -n ingress

NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
default-backend   2         2         2            2           27s

$ kubectl get service

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
appsvc1      ClusterIP   10.98.174.69    <none>        80/TCP    28m
appsvc2      ClusterIP   10.96.193.198   <none>        80/TCP    28m
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   11d

$ kubectl get service -n ingress

NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
default-backend   ClusterIP   10.99.233.157   <none>        80/TCP    38s

$ kubectl get ep -n ingress

NAME              ENDPOINTS                        AGE
default-backend   10.32.0.14:8080,10.40.0.4:8080   2m
```

现在创建一个文件来定义一个 configMap，以便被我们的应用程序使用。

```sh
$ vim nginx-ingress-controller-config-map.yaml
```

```yaml
apiVersion : v1
kind : ConfigMap
metadata :
    name : nginx-ingress-controller-conf
  labels :
      app : nginx-ingress-lb
data :
    enable-vts-status: true
```

然后创建 ConfigMap：

```sh
$ kubectl create -f nginx-ingress-controller-config-map.yaml -n ingress

configmap/nginx-ingress-controller-conf created

$ kubectl get configmaps -n ingress

NAME                            DATA      AGE
nginx-ingress-controller-conf   1         20s

$ kubectl describe configmaps nginx-ingress-controller-conf -n ingress

Name:         nginx-ingress-controller-conf
Namespace:    ingress
Labels:       app=nginx-ingress-lb
Annotations:  <none>
Data
====
enable-vts-status:
----
true
Events:  <none>
```

然后创建关联的 Service Account：

```sh
$ vim nginx-ingress-controller-service-account.yaml
```

```yaml
apiVersion : v1
kind : ServiceAccount
metadata :
    name : nginx
  namespace : ingress
```

```sh
$ vim nginx-ingress-controller-clusterrole.yaml
```

```yaml
kind : ClusterRole
apiVersion : rbac.authorization.k8s.io/v1
metadata :
    name : nginx-role
rules :
- apiGroups :
  - " "
  - " extensions "
   resources :
  - configmaps
  - secrets
  - endpoints
  - ingresses
  - nodes
  - pods
  verbs :
  - list
  - watch
- apiGroups :
  - " "
   resources :
  - services
  verbs :
  - list
  - watch
  - get
  - update
- apiGroups :
  - " extensions "
   resources :
  - ingresses
  verbs :
  - get
- apiGroups :
  - " "
   resources :
  - events
  verbs :
  - create
- apiGroups :
  - " extensions "
   resources :
  - ingresses / status
  verbs :
  - update
- apiGroups :
  - " "
   resources :
  - configmaps
  verbs :
  - get
  - create
```

```sh
$ vim nginx-ingress-controller-clusterrolebinding.yaml
```

```yaml
kind : ClusterRoleBinding
apiVersion : rbac.authorization.k8s.io/v1
metadata :
    name : nginx-role
  namespace : ingress
roleRef :
    apiGroup : rbac.authorization.k8s.io
  kind : ClusterRole
  name : nginx-role
subjects :
- kind : ServiceAccount
  name : nginx
  namespace : ingress
```

然后将这些配置作用到 ingress 命名空间：

```sh
$ kubectl create -f nginx-ingress-controller-service-account.yaml -n ingress

serviceaccount/nginx created

$ kubectl create -f nginx-ingress-controller-clusterrole.yaml -n ingress

clusterrole.rbac.authorization.k8s.io/nginx-role created

$ kubectl create -f nginx-ingress-controller-clusterrolebinding.yaml -n ingress

clusterrolebinding.rbac.authorization.k8s.io/nginx-role created
```

然后创建另一个部署：

```yaml
# vim nginx-ingress-controller-deployment.yaml
apiVersion : apps / v1
kind : Deployment
metadata :
    name : nginx-ingress-controller
spec :
    replicas : 1
  selector :
      matchLabels :
        app : nginx-ingress-lb
  revisionHistoryLimit : 3
  template :
      metadata :
        labels :
          app : nginx-ingress-lb
    spec :
        terminationGracePeriodSeconds : 60
      serviceAccount : nginx
      containers :
        - name : nginx-ingress-controller
          image : quay.io/kubernetes-ingress-controller/nginx-ingress-controllerContact.9.0
          imagePullPolicy : Always
          readinessProbe :
              httpGet :
                path : / healthz
              port : 10254
              scheme : HTTP
          livenessProbe :
              httpGet :
                path : / healthz
              port : 10254
              scheme : HTTP
            initialDelaySeconds: 10
            timeoutSeconds : 5
          args :
            - / nginx-ingress-controller
            - --default-backend-service = ingress/default-backend
            - --configmap = ingress/nginx-ingress-controller-conf
            - --v=2
          env :
            - name : POD_NAME
              valueFrom :
                  fieldRef :
                    fieldPath : metadata.name
            - name : POD_NAMESPACE
              valueFrom :
                  fieldRef :
                    fieldPath : metadata.namespace
          ports :
            - containerPort : 80
            - containerPort : 18080
```

```sh
$ kubectl create -f nginx-ingress-controller-deployment.yaml -n ingress

deployment.apps/nginx-ingress-controller created
```

最后，我们就来定义 Ingress：

```yaml
# vim nginx-ingress.yaml
apiVersion : networking.k8s.io/v1
kind : Ingress
metadata :
    name : nginx-ingress
spec :
    rules :
  - host : ec2-54-198-119-88.compute-1.amazonaws.com # Change to your dns address
    http :
        paths :
      - backend :
            service :
              name: nginx-ingress
            port :
                number : 18080
        path : /nginx_status
        pathType : Prefix
```

现在创建一个文件来定义将重定向到我们在本节开头创建的应用程序的服务的入口。

```yaml
# vim app-ingress.yaml
apiVersion : networking.k8s.io/v1
kind : Ingress
metadata :
    annotations :
      nginx.ingress.kubernetes.io/rewrite-target : /
  name : app-ingress
spec :
    rules :
  - host : ec2-54-198-119-88.compute-1.amazonaws.com # Change to your dns address
    http :
        paths :
      - backend :
            service :
              name : appsvc1
            port :
                number : 80
        path : /app1
        pathType : Prefix
      - backend :
            service :
              name : appsvc2
            port :
                number : 80
        path : /app2
        pathType : Prefix
```

```sh
$ kubectl create -f nginx-ingress.yaml -n ingress

ingress.networking.k8s.io/nginx-ingress created

$ kubectl create -f app-ingress.yaml

ingress.networking.k8s.io/app-ingress created

$ kubectl get ingresses -n ingress

NAME            HOSTS                                        ADDRESS   PORTS     AGE
nginx-ingress   ec2-54-159-116-229.compute-1.amazonaws.com             80        35s

$ kubectl get ingresses

NAME          HOSTS                                        ADDRESS   PORTS     AGE
app-ingress   ec2-54-159-116-229.compute-1.amazonaws.com             80        16s

$ kubectl describe ingresses.extensions nginx-ingress -n ingress

Name:             nginx-ingress
Namespace:        ingress
Address:
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                                        Path  Backends
  ----                                        ----  --------
  ec2-54-159-116-229.compute-1.amazonaws.com
                                              /nginx_status   nginx-ingress:18080 (<none>)
Annotations:
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  50s   nginx-ingress-controller  Ingress ingress/nginx-ingress

$ kubectl describe ingresses.extensions app-ingress

Name:             app-ingress
Namespace:        default
Address:
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                                        Path  Backends
  ----                                        ----  --------
  ec2-54-159-116-229.compute-1.amazonaws.com
                                              /app1   appsvc1:80 (<none>)
                                              /app2   appsvc2:80 (<none>)
Annotations:
  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  1m    nginx-ingress-controller  Ingress default/app-ingress
```

然后我们创建一个 NodePort 服务：

```yaml
# vim nginx-ingress-controller-service.yaml
apiVersion : v1
kind : Service
metadata :
    name : nginx-ingress
spec :
    type : NodePort
  ports :
    - port : 80
      nodePort : 30000
      name : http
    - port : 18080
      nodePort : 32000
      name : http-mgmt
  selector :
      app : nginx-ingress-lb
```

```sh
$ kubectl create -f nginx-ingress-controller-service.yaml -n=ingress

service/nginx-ingress created
```

这样我们就可以直接在外部访问到了：

```sh
$ curl http://SEU-ENDEREÇO:30000/app1

$ curl http://SEU-ENDEREÇO:30000/app2

$ curl http://SEU-ENDEREÇO:32000/nginx_status
```

# 使用外部服务

## 外部 IP

在部分场景下我们需要使用外部的 IP 作为 Ingress 的后端服务，此时我们仍需要创建一个指向外部服务的 Service：

```yml
apiVersion: v1
kind: Service
metadata:
  name: release-name-ufc-rancher
  labels:
    app.kubernetes.io/name: ufc-rancher
    helm.sh/chart: ufc-rancher-0.1.0
    app.kubernetes.io/instance: release-name
    app.kubernetes.io/version: "1.0"
    app.kubernetes.io/managed-by: Tiller
spec:
  type: ClusterIP
  ports:
    - port: 58080
      targetPort: 58080
      protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: release-name-ufc-rancher
subsets:
  - addresses:
      # list all external ips for this service
      - ip: 172.19.157.3
    ports:
      - port: 58080
        protocol: TCP
```

值得注意的是，这里我们在声明服务的时候并未指明 Pod Selector，这也就创建了一个没有后端的 Service，我们需要手动地去创建某个 Endpoints 然后将流量导入到该 Endpoints。外部对服务的访问则是同样创建 Ingress 资源即可：

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: release-name-ufc-rancher
  labels:
    app.kubernetes.io/name: ufc-rancher
    helm.sh/chart: ufc-rancher-0.1.0
    app.kubernetes.io/instance: release-name
    app.kubernetes.io/version: "1.0"
    app.kubernetes.io/managed-by: Tiller
  annotations:
    certmanager.k8s.io/issuer: letsencrypt-prod
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/proxy-body-size: "0"

spec:
  tls:
    - hosts:
        - "k8s.unionfab.com"
      secretName: ufc-rancher-tls
  rules:
    # ufc rancher ingress rules
    - host: "k8s.unionfab.com"
      http:
        paths:
          - path: /
            backend:
              serviceName: release-name-ufc-rancher
              servicePort: 58080
```

这里我们的实例可以参考使用 Ingress 以允许用域名方式访问 Rancher，其 Helm 配置参考 [K8s/Helm](https://github.com/BE-Kits/k8s-examples)。

## 外部域名

# TBD

- https://appscode.com/products/voyager/10.0.0/guides/ingress/http/external-svc/
