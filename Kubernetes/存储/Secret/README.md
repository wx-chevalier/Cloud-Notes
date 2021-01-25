# Secret

Secret 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 ssh key。将这些信息放在 secret 中比放在 Pod 的定义或者 容器镜像 中来说更加安全和灵活。

Secret 是一种包含少量敏感信息例如密码、token 或 key 的对象。这样的信息可能会被放在 Pod spec 中或者镜像中；将其放在一个 secret 对象中可以更好地控制它的用途，并降低意外暴露的风险。

用户可以创建 secret，同时系统也创建了一些 secret。要使用 secret，pod 需要引用 secret。Pod 可以用两种方式使用 secret：作为 volume 中的文件被挂载到 pod 中的一个或者多个容器里，或者当 kubelet 为 pod 拉取镜像时使用。

# TBD

- https://kubernetes.io/zh/docs/concepts/configuration/secret/
