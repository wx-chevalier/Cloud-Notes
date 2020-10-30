# Weave

在每个宿主机上布置一个特殊的 route 的容器，不同宿主机的 route 容器连接起来。route 拦截所有普通容器的 ip 请求，并通过 udp 包发送到其他宿主机上的普通容器。这样在跨机的多个容器端看到的就是同一个扁平网络。weave 解决了网络问题，不过部署依然是单机的。

![image](https://user-images.githubusercontent.com/5803001/45594701-a3127780-b9d1-11e8-8067-6b25fd5a9064.png)
