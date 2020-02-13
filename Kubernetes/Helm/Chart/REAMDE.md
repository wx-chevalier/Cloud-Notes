# Chart

我们创建一个名为 mychart 的 Chart，看一看 Chart 的文件结构。

```sh
$ helm create mongodb
$ tree mongodb
mongodb
├── Chart.yaml #Chart本身的版本和配置信息
├── charts #依赖的chart
├── templates #配置模板目录
│   ├── NOTES.txt # Helm 提示信息
│   ├── _helpers.tpl #用于修改kubernetes objcet配置的模板
│   ├── deployment.yaml #K8s Deployment object
│   └── service.yaml #K8s Serivce
└── values.yaml #K8s object configuration

2 directories, 6 files
```

- Chart.yaml 文件包含 Chart 的描述您可以从模板中访问它。
- template/ 目录用于模板文件，当 Helm 执行 Chart 时，它将通过模板渲染引擎发送 template/ 目录中的所有文件。然后，它将收集这些模板的结果并将其发送到 Kubernetes。
- values.yaml 文件对模板也很重要，该文件包含 Chart 的默认值，用户在 Helm 安装或 Helm 升级期间可能会覆盖这些值。
- charts/ 子目录可能包含其他 Chart（我们称为子 Chart），在本指南的后面，我们将看到模板渲染时它们如何工作。

# 快速开始

## 模板

Templates 目录下是 yaml 文件的模板，遵循 Go template 语法。使用过 Hugo 的静态网站生成工具的人应该对此很熟悉。我们查看下 deployment.yaml 文件的内容。

```yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: {{ template "fullname" . }}
  labels:
    chart: "{{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}"
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    metadata:
      labels:
        app: {{ template "fullname" . }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: {{ .Values.service.internalPort }}
        livenessProbe:
          httpGet:
            path: /
            port: {{ .Values.service.internalPort }}
        readinessProbe:
          httpGet:
            path: /
            port: {{ .Values.service.internalPort }}
        resources:
{{ toyaml .Values.resources | indent 12 }}
```

这是该应用的 Deployment 的 yaml 配置文件，其中的双大括号包扩起来的部分是 Go template，其中的 Values 是在 values.yaml 文件中定义的：

```yml
# Default values for mychart.
# This is a yaml-formatted file.
# Declare variables to be passed into your templates.
replicaCount: 1
image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent
service:
  name: nginx
  type: ClusterIP
  externalPort: 80
  internalPort: 80
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

比如在 Deployment.yaml 中定义的容器镜像 `image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"` 其中的：

- .Values.image.repository 就是 nginx
- .Values.image.tag 就是 stable

以上两个变量值是在 create chart 的时候自动生成的默认值。我们将默认的镜像地址和 tag 改成我们自己的镜像 harbor-001.jimmysong.io/library/nginx:1.9。

## 检查配置和模板是否有效

当使用 K8s 部署应用的时候实际上讲 templates 渲染成最终的 K8s 能够识别的 yaml 格式。使用 `helm install --dry-run --debug <chart_dir>` 命令来验证 chart 配置。该输出中包含了模板的变量配置与最终渲染的 yaml 文件。

```yml
$ helm install --dry-run --debug mychart
Created tunnel using local port: '58406'
SERVER: "localhost:58406"
CHART PATH: /Users/jimmy/Workspace/github/bitnami/charts/incubator/mean/charts/mychart
NAME:   filled-seahorse
REVISION: 1
RELEASED: Tue Oct 24 18:57:13 2017
CHART: mychart-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
image:
  pullPolicy: IfNotPresent
  repository: harbor-001.jimmysong.io/library/nginx
  tag: 1.9
replicaCount: 1
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
service:
  externalPort: 80
  internalPort: 80
  name: nginx
  type: ClusterIP

HOOKS:
MANIFEST:

---
# Source: mychart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: filled-seahorse-mychart
  labels:
    chart: "mychart-0.1.0"
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: nginx
  selector:
    app: filled-seahorse-mychart

---
# Source: mychart/templates/deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: filled-seahorse-mychart
  labels:
    chart: "mychart-0.1.0"
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: filled-seahorse-mychart
    spec:
      containers:
      - name: mychart
        image: "harbor-001.jimmysong.io/library/nginx:1.9"
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
        resources:
            limits:
              cpu: 100m
              memory: 128Mi
            requests:
              cpu: 100m
              memory: 128Mi
```

我们可以看到 Deployment 和 Service 的名字前半截由两个随机的单词组成，最后才是我们在 values.yaml 中配置的值。

## 部署到 Kubernetes

在 mychart 目录下执行下面的命令将 nginx 部署到 K8s 集群上。

```yml
helm install .
NAME:   eating-hound
LAST DEPLOYED: Wed Oct 25 14:58:15 2017
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                  CLUSTER-IP     EXTERNAL-IP  PORT(S)  AGE
eating-hound-mychart  10.254.135.68  <none>       80/TCP   0s

==> extensions/v1beta1/Deployment
NAME                  DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
eating-hound-mychart  1        1        1           0          0s


NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=eating-hound-mychart" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:80
```

现在 Nginx 已经部署到 K8s 集群上，本地执行提示中的命令在本地主机上访问到 Nginx 实例。

```sh
$ export POD_NAME=$(kubectl get pods --namespace default -l "app=eating-hound-mychart" -o jsonpath="{.items[0].metadata.name}")

$ echo "Visit http://127.0.0.1:8080 to use your application"

$ kubectl port-forward $POD_NAME 8080:80
```

在本地访问 `http://127.0.0.1:8080` 即可访问到 Nginx

# NOTES.txt

在 `chart install`或`chart upgrade` 结束时，Helm 可以为用户打印出一大堆有用的信息。这些信息是使用模板高度定制的。要将安装说明添加到 chart，只需创建一个 `templates/NOTES.txt` 文件即可。这个文件是纯文本的，但是它像一个模板一样处理，并且具有所有可用的普通模板函数和对象。我们来创建一个简单的 `NOTES.txt` 文件：

```
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get {{ .Release.Name }}
```

现在，如果我们运行 `helm install ./mychart` 我们会在底部看到这条消息：

```
RESOURCES:
==> v1/Secret
NAME                   TYPE      DATA      AGE
rude-cardinal-secret   Opaque    1         0s

==> v1/ConfigMap
NAME                      DATA      AGE
rude-cardinal-configmap   3         0s


NOTES:
Thank you for installing mychart.

Your release is named rude-cardinal.

To learn more about the release, try:

  $ helm status rude-cardinal
  $ helm get rude-cardinal
```

使用`NOTES.txt`这种方式是一种很好的方式，可以为用户提供有关如何使用新安装 chart 的详细信息。强烈建议创建一个文件`NOTES.txt`，尽管这不是必需的。
