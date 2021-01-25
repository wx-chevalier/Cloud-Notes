# Hooks

Helm 提供了一个 hook 机制，允许 chart 开发人员在 release 的生命周期中的某些点进行干预。例如，可以使用 hooks 来：

- 在加载任何其他 chart 之前，在安装过程中加载 ConfigMap 或 Secret。
- 在安装新 chart 之前执行作业以备份数据库，然后在升级后执行第二个作业以恢复数据。
- 在删除 release 之前运行作业，以便在删除 release 之前优雅地停止服务。

Hooks 像常规模板一样工作，但它们具有特殊的注释，可以使 Helm 以不同的方式使用它们。

# 可用的 Hooks

定义了以下 hooks：

- 预安装 pre-install:：在模板渲染后执行，但在 Kubernetes 中创建任何资源之前执行。
- 安装后 post-install：在所有资源加载到 Kubernetes 后执行
- 预删除 pre-delete：在从 Kubernetes 删除任何资源之前执行删除请求。
- 删除后 post-delete：删除所有 release 的资源后执行删除请求。
- 升级前 pre-upgrade：在模板渲染后，但在任何资源加载到 Kubernetes 之前执行升级请求（例如，在 Kubernetes 应用操作之前）。
- 升级后 post-upgrade：在所有资源升级后执行升级。
- 预回滚 pre-rollback：在渲染模板之后，但在任何资源已回滚之前，在回滚请求上执行。
- 回滚后 post-rollback：在修改所有资源后执行回滚请求。

# Hook 声明

Hook 只是 Kubernetes manifest 文件，在 metadata 部分有特殊的注释 。因为他们是模板文件，可以使用所有的 Normal 模板的功能，包括读取 .Values，.Release 和 .Template。例如，在此模板中, 存储在 templates/post-install-job.yaml 的声明要在 post-install 阶段运行作业：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{.Release.Name}}"
  labels:
    app.kubernetes.io/managed-by: {{.Release.Service | quote}}
    app.kubernetes.io/instance: {{.Release.Name | quote}}
    helm.sh/chart: "{{.Chart.Name}}-{{.Chart.Version}}"
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      name: "{{.Release.Name}}"
      labels:
      app.kubernetes.io/managed-by: {{.Release.Service | quote}}
      app.kubernetes.io/instance: {{.Release.Name | quote}}
      helm.sh/chart: "{{.Chart.Name}}-{{.Chart.Version}}"
    spec:
      restartPolicy: Never
      containers:
      - name: post-install-job
        image: "alpine:3.3"
        command: ["/bin/sleep","{{default"10".Values.sleepyTime}}"]
```

注释使这个模板成为 hook：

```
  annotations:
    "helm.sh/hook": post-install
```

一个资源可以部署多个 hook：

```
  annotations:
    "helm.sh/hook": post-install,post-upgrade
```

同样，实现一个给定的 hook 的不同种类资源数量没有限制。例如，我们可以将 secret 和 config map 声明为预安装 hook。

子 chart 声明 hook 时，也会评估这些 hook。顶级 chart 无法禁用子 chart 所声明的 hook。

可以为一个 hook 定义一个权重，这将有助于建立一个确定性的执行顺序。权重使用以下注释来定义：

```
  annotations:
    "helm.sh/hook-weight": "5"
```

hook 权重可以是正数或负数，但必须表示为字符串。当 Tiller 开始执行一个特定类型的 hook (例：`pre-install` hooks `post-install` hooks, 等等) 执行周期时，它会按升序对这些 hook 进行排序。

还可以定义确定何时删除相应的 hook 资源的策略。hook 删除策略使用以下注释来定义：

```
  annotations:
    "helm.sh/hook-delete-policy": hook-succeeded
```

可以选择一个或多个定义的注释值：

- "hook-succeeded" 指定 Tiller 应该在 hook 成功执行后删除 hook。
- "hook-failed" 指定如果 hook 在执行期间失败，Tiller 应该删除 hook。
- "before-hook-creation" 指定 Tiller 应在删除新 hook 之前删除以前的 hook。
