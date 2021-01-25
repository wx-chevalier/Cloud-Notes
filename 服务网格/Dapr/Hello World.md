# Dapr Hello World

# 环境安装

Dapr 服务端的安装还是非常方便的，首先我们需要安装 Dapr 命令行：

```sh
# Linux
$ wget -q https://raw.githubusercontent.com/dapr/cli/master/install/install.sh -O - | /bin/bash

# Mac
$ curl -fsSL https://raw.githubusercontent.com/dapr/cli/master/install/install.sh | /bin/bash
```

然后运行 Daprd 服务程序：

```sh
$ dapr init
⌛  Making the jump to hyperspace...
Downloading binaries and setting up components
✅  Success! Dapr is up and running. To get started, go here: https://aka.ms/dapr-getting-started
```

接下来可以指定运行时：

```sh
# Install v0.1.0 runtime
$ dapr init --runtime-version 0.1.0

# Check the versions of cli and runtime
$ dapr --version
cli version: v0.1.0
runtime version: v0.1.0
```

# 应用运行

接下来将演示如何使 Dapr 在您的计算机上本地运行。您将部署一个 Node.js 应用，该应用订阅订购消息并保留它们。以下架构图说明了组成第一部分样本的组件：

![Node.js Application](https://s1.ax1x.com/2020/09/22/wORITP.png)

稍后，您将部署一个 Python 应用程序以充当发布者。下面的架构图显示了新组件的添加：

![Python Application](https://s1.ax1x.com/2020/09/22/wORqSg.png)

## Node.js 代码

在 Node.js 中就是简单的 Express 应用：

```js
const daprPort = process.env.DAPR_HTTP_PORT || 3500;
const stateStoreName = `statestore`;
const stateUrl = `http://localhost:${daprPort}/v1.0/state/${stateStoreName}`;
```

Dapr CLI 为 Dapr 端口创建一个环境变量，默认为 3500。向系统发送 POST 消息时，将在第 3 步中使用它。stateStoreName 是提供给状态存储的名称。稍后您将返回到该名称，以了解该名称的配置方式。

```js
app.post("/neworder", (req, res) => {
  const data = req.body.data;
  const orderId = data.orderId;
  console.log("Got a new order! Order ID: " + orderId);

  const state = [
    {
      key: "order",
      value: data,
    },
  ];

  fetch(stateUrl, {
    method: "POST",
    body: JSON.stringify(state),
    headers: {
      "Content-Type": "application/json",
    },
  })
    .then((response) => {
      if (!response.ok) {
        throw "Failed to persist state.";
      }

      console.log("Successfully persisted state.");
      res.status(200).send();
    })
    .catch((error) => {
      console.log(error);
      res.status(500).send({ message: error });
    });
});
```

该应用程序在这里公开了将接收和处理新订单消息的终结点。它首先记录传入的消息，然后通过将状态数组发布到 `/state/<state-store-name>` 端点来将订单 ID 持久化到 Redis 或者，您可以通过简单地将其与响应对象一起返回来保持状态：

```js
res.json({
  state: [
    {
      key: "order",
      value: order,
    },
  ],
});
```
