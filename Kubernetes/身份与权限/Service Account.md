# Service Account

Service Account 为 Pod 中的进程提供身份信息。当用户访问集群（例如使用 kubectl 命令）时，apiserver 会将您认证为一个特定的 User Account（目前通常是 admin，除非您的系统管理员自定义了集群配置）。Pod 容器中的进程也可以与 apiserver 联系。 当它们在联系 apiserver 的时候，它们会被认证为一个特定的 Service Account（例如 default）。

# 使用默认的 Service Account 访问 API server

当您创建 pod 的时候，如果您没有指定一个 service account，系统会自动得在与该 pod 相同的 namespace 下为其指派一个 default service account。如果您获取刚创建的 pod 的原始 json 或 yaml 信息（例如使用 kubectl get pods/podename -o yaml 命令），您将看到 spec.serviceAccountName 字段已经被设置为 default。

您可以在 pod 中使用自动挂载的 service account 凭证来访问 API，如 Accessing the Cluster 中所描述。Service account 是否能够取得访问 API 的许可取决于您使用的 授权插件和策略。在 1.6 以上版本中，您可以选择取消为 service account 自动挂载 API 凭证，只需在 service account 中设置 automountServiceAccountToken: false：

```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
automountServiceAccountToken: false
# ...
```

在 1.6 以上版本中，您也可以选择只取消单个 pod 的 API 凭证自动挂载：

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: build-robot
  automountServiceAccountToken: false
#  ...
```

如果在 pod 和 service account 中同时设置了 automountServiceAccountToken , pod 设置中的优先级更高。

# 使用多个 Service Account
