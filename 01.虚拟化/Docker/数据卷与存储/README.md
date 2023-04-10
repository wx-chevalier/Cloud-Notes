# Docker 数据卷与存储

在我们使用 Docker 容器时，为了持久化数据或多容器共享数据，我们可以在 Dockerfile 中通过 VOLUME 关键字声明容器要持久化数据的目录列表，或者通过：

```sh
$ docker create/run --name containername --volume volumename ...
```

在创建或运行容器时添加存储卷，以上两种方式创建的 volume 在 docker 中叫做 anonymous volume, 主要用在 volume 不需要多容器共享的场景(当然也可以通过--volumes-from 参数来共享)。而为了在容器层面对接不同的存储系统，docker 定义了一套 volume plugin 机制。简单来说就是 docker engine 抽象了一套 REST API 并作为 client 端，各种存储系统通过实现该套 API 作为 server 端来对接真正的存储系统，然后可以通过下面的命令来使用：

```sh
# 通过driver指定创建volume的类型，opt指定创建存储所需参数，docker engine会通过driver类型将创建存储的请求forward到同node上的相应的volume plugin来真实执行创建存储的动作
$ docker volume create --driver=xxx volumename --opt key1=value1 --opt key2=value2
$ docker run -it --name container1 --volume volumename:/data busybox sh
$ docker run -it --name container2 --volume volumename:/data ubuntu sh
```

上面这种方式创建的 volume 也叫做 named volume，volume 的创建和容器的创建是解耦的，在多容器共享存储时语义更清晰明了，而且可以插件机制来丰富持久化存储的存储类型。
