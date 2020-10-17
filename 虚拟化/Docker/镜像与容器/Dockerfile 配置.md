# Dockerfile 配置

# 安全配置

容器安全是一个广泛的问题空间，有很多低垂的果实可以收获来降低风险。一个好的出发点是在编写 Dockerfiles 时遵循一些规则。

## Do not store secrets in environment variables

密钥分发是一个棘手的问题，而且很容易做错。对于容器化的应用，人们可以通过挂载卷从文件系统中浮出水面，或者通过环境变量更方便地浮出水面。使用 ENV 来存储密钥是不好的做法，因为 Dockerfiles 通常是和应用一起分发的，所以和在代码中硬编码密钥没有区别。

```sh
secrets_env = [
    "passwd",
    "password",
    "pass",
 #  "pwd", can't use this one
    "secret",
    "key",
    "access",
    "api_key",
    "apikey",
    "token",
    "tkn"
]

deny[msg] {
    input[i].Cmd == "env"
    val := input[i].Value
    contains(lower(val[_]), secrets_env[_])
    msg = sprintf("Line %d: Potential secret in ENV key found: %s", [i, val])
}
```

## 仅使用可信镜像

容器化应用的供应链攻击也将来自用于构建容器本身的层级。主要的罪魁祸首显然是使用的基础映像。不受信任的基础镜像是一种高风险，应尽可能避免使用。Docker 为大多数常用的操作系统和应用程序提供了一套官方基础镜像。通过使用它们，我们利用与 Docker 本身的某种责任分担，最大限度地降低了泄露的风险。

```sh
deny[msg] {
    input[i].Cmd == "from"
    val := split(input[i].Value[0], "/")
    count(val) > 1
    msg = sprintf("Line %d: use a trusted base image", [i])
}
```

这个规则是针对 DockerHub 的官方镜像调整的，信任的定义取决于你的上下文，相应地修改这个规则。

## 避免使用 latest 标签

锁定基础镜像的版本，可以让你对你正在构建的容器的可预测性放心一些。如果你依赖最新版本，你可能会默默地继承更新的包，在最好的最坏的情况下可能会影响你的应用程序的可靠性，在最坏的最坏的情况下可能会引入一个漏洞。

```sh
deny[msg] {
    input[i].Cmd == "from"
    val := split(input[i].Value[0], ":")
    contains(lower(val[1]), "latest"])
    msg = sprintf("Line %d: do not use 'latest' tag for base images", [i])
}
```

## 避免使用 curl

从互联网上拉东西，然后用管道输送到 shell 中，这是最糟糕的。不幸的是，这是一个普遍的解决方案，以简化软件的安装。

```sh
$ wget https://cloudberry.engineering/absolutely-trustworthy.sh | sh
```

供应链攻击的风险是一样的框架，归根结底是信任。如果你真的要使用 curl bash，请正确使用。使用一个可信的源头、安全连接、验证您所下载内容的真实性和完整性。

```sh
deny[msg] {
    input[i].Cmd == "run"
    val := concat(" ", input[i].Value)
    matches := regex.find_n("(curl|wget)[^|^>]*[|>]", lower(val), -1)
    count(matches) > 0
    msg = sprintf("Line %d: Avoid curl bashing", [i])
}
```

## 不要升级系统包

这可能有点牵强，但道理如下：你要将软件依赖的版本固定下来，如果你进行 apt-get 升级，你将有效地将它们全部升级到最新版本。如果你确实升级了，而且你使用最新的标签作为基础镜像，你就会放大你的依赖树的不可预测性。你要做的是将基础镜像的版本钉在 apt/apk 更新上。

```sh
upgrade_commands = [
    "apk upgrade",
    "apt-get upgrade",
    "dist-upgrade",
]

deny[msg] {
    input[i].Cmd == "run"
    val := concat(" ", input[i].Value)
    contains(val, upgrade_commands[_])
    msg = sprintf(“Line: %d: Do not upgrade your system packages", [i])
}
```

## 避免使用 ADD

ADD 命令有一个小特点，就是你可以把它指向一个远程的 url，它就会在构建的时候获取内容。

```sh
ADD https://cloudberry.engineering/absolutely-trust-me.tar.gz
```

具有讽刺意味的是，官方文档建议使用 curl bashing 代替。从安全角度来看，同样的建议也适用：不要。先获取你需要的任何内容，验证后再复制。但如果你真的必须这样做，请通过安全连接使用可信的来源。

```sh
deny[msg] {
    input[i].Cmd == "add"
    msg = sprintf("Line %d: Use COPY instead of ADD", [i])
}
```

## 不要使用 root

容器中的 root 和主机上的 root 是一样的，但是受到 docker 守护进程配置的限制。不管有什么限制，如果一个行为者突破了容器，他仍然能够找到一种方法来获得对主机的完全访问。当然这并不理想，你的威胁模型不能忽视以 root 身份运行所带来的风险。因此最好总是指定一个用户。

```sh
USER hopefullynotroot
```

请注意，在 Dockerfile 中明确设置用户只是一层防御，并不能解决整个以 root 身份运行的问题。相反，人们可以采取深度防御的方法，并在整个堆栈中进一步缓解：严格配置 docker 守护进程或使用无根容器解决方案，限制运行时的配置（如果可能的话禁止 --privileged，等等），等等。

```sh
any_user {
    input[i].Cmd == "user"
 }

deny[msg] {
    not any_user
    msg = "Do not run as root, use USER instead"
}
```

## 不要使用 sudo

```sh
deny[msg] {
    input[i].Cmd == "run"
    val := concat(" ", input[i].Value)
    contains(lower(val), "sudo")
    msg = sprintf("Line %d: Do not use 'sudo' command", [i])
}
```
