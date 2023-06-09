# Docker & Kubernetes 小记

# 第一章：Docker 简介与安装

- **Docker 是什么？**

Docker 是一个**生态**，这个生态专注于管理 Containers（容器）。

- **为什么用 Docker？**

Docker 使得安装软件与运行软件十分便利。

安装软件时，可能会碰到各种各样的问题，解决这些问题可能会花费很多人力物力。而 Docker 就是想标准化“安装流程”，让用户直接进入运行软件步骤，节省用户成本。

课程中使用安装 Redis 作为例子，来展示普通安装和使用 Docker 启动的效率差别。

- **基础概念**

1. Image

   Image 也称为**镜像**，是一个包含运行指定程序的文件系统快照和启动命令的文件。通常在 Docker Hub（https://hub.docker.com/） 下载。

2. Container

   Container 也就是**容器**，是 Image 的一个实例。是一个正在运行的进程，以及机器物理资源的子集。

   不同的 Containers 之间，资源是隔离的。

3. Docker Client

   用户通过终端与 Docker Client 交互，一个解析用户命令的程序，并发送给 Docker Server。

   这个程序本身没有处理 Image 与 Container 的功能。

4. Docker Server

   也称之为 Docker Daemon，是负责创建、运行容器的程序。

- **安装**

官方下载网站：https://docs.docker.com/get-docker/

注册后安装对应版本。

⚠️ 注意：安装同时包含了 Linux 虚拟机

- **docker version**

安装完成后，在终端输入 **docker version**，如果输出相关信息说明安装成功了。

- **启动第一个容器**

在终端输入 **docker run hello-world**，之后终端会输出如下信息：

```shell
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
latest: Pulling from library/hello-world
7050e35b49f5: Pull complete
Digest: sha256:94ebc7edf3401f299cd3376a1669bc0a49aef92d6d2669005f9bc5ef028dc333
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

hello-world 是 Docker 社区维护的一个官方镜像，作用就是演示 Docker 化。

Docker Client 收到终端发来的命令后，检查处理后发送给 Docker Server 来创建容器。创建时会先在本地查找镜像缓存，如果缓存中没有，就会去 Docker Hub 上下载镜像后再继续。

# 第二章：了解 Docker Client 中基本命令

本章通过学习 8 个命令的基础使用方式来熟悉 Docker Client。

- **docker run**

第一章启动容器时我们使用了 **docker run** 命令，这个命令会先根据镜像中的快照创建容器，然后再用镜像中的启动命令启动容器。

该命令的使用格式如下：

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

除了第一章中的用法，还可以在镜像名称后，[COMMAND] 处添加自定义启动命令：

```shell
wuxianmimi ~ % docker run busybox ls
bin
dev
etc
home
lib
lib64
proc
root
sys
tmp
usr
var
```

echo 信息：

```shell
wuxianmimi ~ % docker run busybox echo wuxianmimi
wuxianmimi
wuxianmimi ~ %
```

⚠️：使用自定义启动命令后会覆盖默认启动命令。

上面的例子使用的是 busybox 镜像，但如果我们用 hello-world 镜像来使用相同的启动命令，会报错：

```shell
wuxianmimi ~ % docker run hello-world ls
docker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "ls": executable file not found in $PATH: unknown.
wuxianmimi ~ %
```

原因就是 hello-world 镜像中的文件系统快照，并没有执行 ls 的程序。我们传递所有启动命令，必须得在镜像的文件系统快照中存在对应的程序才行。

⚠️：docker run = docker create + docker start

- **docker ps**

**docker ps** 的格式如下：

```shell
docker ps [OPTIONS]
```

这个命令可以输出计算机上所有正在运行的容器列表，先前测试的两个容器都是立即执行并退出的，所以我们得到列表为空：

```shell
wuxianmimi ~ % docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
wuxianmimi ~ %
```

为了测试 **docker ps**，我们可以先在终端输入命令：

```shell
docker run busybox ping bilibili.com
```

然后新建终端窗口，输入 **docker ps**，会得到以下结果：

```shell
wuxianmimi ~ % docker ps
CONTAINER ID   IMAGE     COMMAND               CREATED         STATUS         PORTS     NAMES
35baf22ff8c1   busybox   "ping bilibili.com"   7 seconds ago   Up 6 seconds             hardcore_chatelet
wuxianmimi ~ %
```

如果要查看所有的容器，而不只是正在运行的，可以添加 --**all** 选项：

```shell
docker ps --all
```

- **docker start**

使用 **docker ps --all** 列出所有容器后，我们发现之前运行 hello-world 的容器目前处在 Exited 状态：

```shell
wuxianmimi ~ % docker ps --all
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS                      PORTS     NAMES
cc5bff91b03c   hello-world              "/hello"                 3 seconds ago    Exited (0) 2 seconds ago              pedantic_hugle
wuxianmimi ~ %
```

在 Exited 状态并不代表容器没用了，我们可以使用 **docker start** 命令再次运行容器：

```shell
docker start -a cc5bff91b03c
```

“cc5bff91b03c”是容器的 CONTAINER ID，选项 **-a** 来获取容器的标准输出/标准错误。

**docker start** 的命令格式：

```shell
docker start [OPTIONS] CONTAINER [CONTAINER...]
```

- **docker system prune**

这个命令会一口气删除所有未运行的容器来节省空间，同时也会删除本地的镜像缓存。

- **docker logs**

如果我们在启动容器（**docker start**）时，忘记添加 **-a** 选项怎么办？我们就不能获得容器的标准输出了。

docker logs 命令就是可以在不重新启动容器的情况下，获取容器的标准输出。

命令格式：

```shell
docker logs [OPTIONS] CONTAINER
```

示例：

```shell
wuxianmimi ~ % docker create busybox echo hi mimi
6158cf3c32a190ff0bcb946d63317960269adbe70e71d8480022c82c0897ce74
wuxianmimi ~ % docker start 6158cf3c32a190ff0bcb946d63317960269adbe70e71d8480022c82c0897ce74
6158cf3c32a190ff0bcb946d63317960269adbe70e71d8480022c82c0897ce74
wuxianmimi ~ % docker ps --all
CONTAINER ID   IMAGE     COMMAND          CREATED          STATUS                     PORTS     NAMES
6158cf3c32a1   busybox   "echo hi mimi"   22 seconds ago   Exited (0) 8 seconds ago             affectionate_dirac
wuxianmimi ~ % docker logs 6158cf3c32a1
hi mimi
wuxianmimi ~ %
```

- **停止容器**

有两种方法可以停止容器，**docker stop** 和 **docker kill**。

**docker stop** 发送 SIGTERM 信号给容器，容器接收到信号后，执行相关逻辑并停止。

```shell
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

**docker kill** 发送 SIGKILL 信号给容器，使它立即终止。

```shell
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```

一般情况下我们使用 **docker stop** 来停止容器，对于容器已经未响应的情况，再使用 **docker kill** 命令。不过默认情况下，**docker stop** 命令会在信号发送后 10 秒钟后，对未停止的容器再发送 SIGKILL 信号。

- **docker exec**

课程中通过 redis 演示，在安装了 redis 的机器上运行 redis-server 启动服务，然后在另一个终端输入 redis-cli 来连接 redis 服务。

然而用 **docker run redis** 启动 redis-server 后，我们如何连接这个服务？

这就要使用到 **docker exec**：

```shell
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

这个命令使得我们可以在运行中的容器内执行命令：

```shell
wuxianmimi ~ % docker ps --all
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS                       PORTS     NAMES
19b3b19d3225   redis     "docker-entrypoint.s…"   About a minute ago   Exited (1) 13 seconds ago              vigorous_williams
a9cd082a2d4c   busybox   "ping bilibili.com"      9 minutes ago        Exited (137) 9 minutes ago             heuristic_hopper
6158cf3c32a1   busybox   "echo hi mimi"           23 minutes ago       Exited (0) 23 minutes ago              affectionate_dirac
wuxianmimi ~ % docker start 19b3b19d3225
19b3b19d3225
wuxianmimi ~ % docker exec -it 19b3b19d3225 redis-cli
127.0.0.1:6379> set myname mimi
OK
127.0.0.1:6379> get myname
"mimi"
127.0.0.1:6379>
```

在上面的例子中，我们先启动了 redis 容器，然后通过命令 **docker exec -it [容器 ID] redis-cli** 成功在容器内执行了命令，连接到 redis-server 中。

命令中的 **-it** 有什么作用？

**-it** 是简写，相当于 **-i -t**。**-i**（-interactive） 选项使得我们终端保持连接到容器的标准输入，如果不添加的话，我们无法执行 redis-cli 连接成功后的操作。而 **-t**（-tty） 选项，可以理解为对容器 3 个标准的文件描述符的格式化，使得用户交互更方便。

不加 **-it** 选项执行 **docker exec** 命令，终端上仍旧没地方输入：

```shell
wuxianmimi ~ % docker exec 19b3b19d3225 redis-cli
wuxianmimi ~ %
```

只加 **-i** 选项执行 **docker exec** 命令，输入命令时没了提示，且输出的格式没之前好看：

```shell
wuxianmimi ~ % docker exec -i 19b3b19d3225 redis-cli
set myname mimi
OK
get myname
mimi
```

如果我们要在容器内执行很多命令，那么一直反复的敲 **docker exec** 我们也手累，这时候可以输入：

```shell
docker exec -it 19b3b19d3225 sh
```

我们会在容器内打开 sh，sh 是 shell，可以让我们可以在容器内执行命令（相当于进入了一台 Linux 虚拟机）

```shell
wuxianmimi ~ % docker exec -it 19b3b19d3225 sh
# cd /
# ls
bin  boot  data  dev  etc  home  lib  media  mnt  opt  proc  root  run	sbin  srv  sys	tmp  usr  var
# redis-cli
127.0.0.1:6379>
#
```

需要退出时，使用 Ctrl+d 快捷键。

⚠️：docker run 也可以添加 -it 选项

# 第三章：了解 Dockerfile

基本上围绕着如何书写 Dockerfile，以实现一个 redis-server 为例。

- **Dockerfile**

我们创建自定义镜像从 Dockerfile 开始。

Dockerfile 是一个文本文件，在里面添加我们的配置信息，这些信息定义了容器的行为方式。

Dockerfile 文件经由 Docker Client 传给 Docker Server，最后由 Docker Server 解析并构建镜像。

Dockerfile 中添加的配置项一般是三步曲（工作流）

1. 规定**基础镜像**
2. 运行命令来**安装**必要的依赖、程序
3. 规定容器**启动参数**

创建一个文件夹，并在文件夹内创建 Dockerfile 文件，内容如下：

```shell
# 三步曲1：规定基础镜像
# 基础镜像可以理解为操作系统，我们在安装其他程序前，需要先安装操作系统
FROM alpine

# 三步曲2：运行命令来安装必要的依赖、程序
# apk 是 alpine 内建的包管理命令，我们使用这个包管理程序安装 redis
RUN apk add --update redis

# 三步曲3：规定容器启动参数
CMD ["redis-server"]
```

在文件夹根目录内，使用 **docker build .** 命令（不要漏了最后的 .）：

```shell
wuxianmimi docker-redis-image % docker build .
[+] Building 15.5s (7/7) FINISHED
 => [internal] load build definition from Dockerfile                                                                                                              0.0s
 => => transferring dockerfile: 240B                                                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                                                 0.0s
 => => transferring context: 2B                                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                                                                  5.1s
 => [1/2] FROM docker.io/library/alpine@sha256:8914eb54f968791faf6a8638949e480fef81e697984fba772b3976835194c6d4                                                   5.8s
 => => resolve docker.io/library/alpine@sha256:8914eb54f968791faf6a8638949e480fef81e697984fba772b3976835194c6d4                                                   0.0s
 => => sha256:8914eb54f968791faf6a8638949e480fef81e697984fba772b3976835194c6d4 1.64kB / 1.64kB                                                                    0.0s
 => => sha256:af06af3514c44a964d3b905b498cf6493db8f1cde7c10e078213a89c87308ba0 528B / 528B                                                                        0.0s
 => => sha256:d3156fec8bcbc7b491a4edc271a7734dcfa186fc73282d4e120eeaaf2ce95c43 1.49kB / 1.49kB                                                                    0.0s
 => => sha256:261da4162673b93e5c0e7700a3718d40bcc086dbf24b1ec9b54bca0b82300626 3.26MB / 3.26MB                                                                    5.6s
 => => extracting sha256:261da4162673b93e5c0e7700a3718d40bcc086dbf24b1ec9b54bca0b82300626                                                                         0.1s
 => [auth] library/alpine:pull token for registry-1.docker.io                                                                                                     0.0s
 => [2/2] RUN apk add --update redis                                                                                                                              4.6s
 => exporting to image                                                                                                                                            0.0s
 => => exporting layers                                                                                                                                           0.0s
 => => writing image sha256:5159f921568f4b843c002a76d29c37ca0ffbef1d284352a7b50691dd29d3fe29                                                                      0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
```

5159f921568f4b843c002a76d29c37ca0ffbef1d284352a7b50691dd29d3fe29，这一串就是我们镜像的 ID。

Dockerfile 中的每条指令都会“返回”一个镜像，在基于这个镜像创建的临时容器中执行下一条指令，临时容器在指令执行完毕后会被删除，而这个容器的文件系统快照会被保存到当前指令“返回”的镜像中。

使用 **docker run** 命令创建并运行容器：

```shell
wuxianmimi docker-redis-image % docker run 5159f921568f4b843c002a76d29c37ca0ffbef1d284352a7b50691dd29d3fe29
1:C 07 Jan 2023 13:29:48.051 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 07 Jan 2023 13:29:48.051 # Redis version=7.0.7, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 07 Jan 2023 13:29:48.051 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1:M 07 Jan 2023 13:29:48.052 * monotonic clock: POSIX clock_gettime
1:M 07 Jan 2023 13:29:48.052 * Running mode=standalone, port=6379.
1:M 07 Jan 2023 13:29:48.052 # Server initialized
1:M 07 Jan 2023 13:29:48.054 * Ready to accept connections
```

到这步，我们创建的第一个镜像就顺利完工了～～

但是现在有个问题，我们每次运行都要粘贴这一长串镜像 ID 太麻烦了，有没有什么好办法解决这个问题？

答案就是在 **docker build** 命令增加 **-t** 选项来给我们自定义镜像加标签：

```shell
wuxianmimi docker-redis-image % docker build -t wuxianmimi/redis:latest .
[+] Building 1.6s (6/6) FINISHED
 => [internal] load build definition from Dockerfile                                                                              0.0s
 => => transferring dockerfile: 37B                                                                                               0.0s
 => [internal] load .dockerignore                                                                                                 0.0s
 => => transferring context: 2B                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                                  1.5s
 => [1/2] FROM docker.io/library/alpine@sha256:8914eb54f968791faf6a8638949e480fef81e697984fba772b3976835194c6d4                   0.0s
 => CACHED [2/2] RUN apk add --update redis                                                                                       0.0s
 => exporting to image                                                                                                            0.0s
 => => exporting layers                                                                                                           0.0s
 => => writing image sha256:5159f921568f4b843c002a76d29c37ca0ffbef1d284352a7b50691dd29d3fe29                                      0.0s
 => => naming to docker.io/wuxianmimi/redis:latest                                                                                0.0s
```

标签的约定格式就是： **[你的 DockerID]/[镜像名]:[版本号]**

# 第四章：构建自定义镜像（NodeJS Demo）

这一章演示了如何将一个 NodeJS Web 应用打包成镜像，并使用该镜像运行容器，最后可以在浏览器访问这个 Web 应用。

新建文件夹，并在文件夹内创建两个文件：

package.json

```javascript
{
  "name": "docker-simpleweb",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start":  "node index.js"
  },
  "keywords": [],
  "author": "wuxianmimi",
  "license": "ISC",
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

index.js

```javascript
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.send("你好鸭");
});

app.listen(8080, () => {
  console.log("已监听8080端口");
});
```

- **基础镜像问题**

创建 Dockerfile 文件，并增加 3 条指令：

```shell
# 三步曲1：规定基础镜像
FROM alpine

# 三步曲2：运行命令来安装必要的依赖、程序
RUN npm install

# 三步曲3：规定容器启动参数
CMD ["npm", "start"]
```

然后运行 **docker build .** 命令，但是我们在第二步安装依赖时会报错：

```shell
wuxianmimi docker-simpleweb % docker build .
[+] Building 2.3s (5/5) FINISHED
 => [internal] load build definition from Dockerfile                                                                              0.0s
 => => transferring dockerfile: 229B                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                 0.0s
 => => transferring context: 2B                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                                  2.1s
 => CACHED [1/2] FROM docker.io/library/alpine@sha256:8914eb54f968791faf6a8638949e480fef81e697984fba772b3976835194c6d4            0.0s
 => ERROR [2/2] RUN npm install                                                                                                   0.2s
------
 > [2/2] RUN npm install:
#5 0.172 /bin/sh: npm: not found
------
executor failed running [/bin/sh -c npm install]: exit code: 127
```

报错说没有找到 npm 这个命令，原因就是 alpine 这个镜像中，并没有 NodeJS 环境，当然也不能运行 npm 命令。

我们当然可以在使用 apt 自己安装 NodeJS 环境，但是 Docker 官方提供了 Node 基础镜像，镜像中已经安装了 Node 开发需要的工具，所以我们直接使用即可。

Docker Hub 上 Node 镜像地址：https://hub.docker.com/_/node

在 Supported tags and respective Dockerfile links 一栏下发现有很多支持的镜像标签，这里咪咪建议使用 14-alpine 这个标签，原因是 NodeJs 16 版本改动较大，可能与一些框架的老版本存在兼容性问题。

视频中讲师使用的 NodeJs 大版本是 8，喜欢原汁原味的同学可以使用这个版本。不过这个版本咪咪没有测试过，不一定能走通后续的流程。

```shell
# 三步曲1：规定基础镜像
# alpine ===> node:14-alpine
FROM node:14-alpine

# 三步曲2：运行命令来安装必要的依赖、程序
RUN npm install

# 三步曲3：规定容器启动参数
CMD ["npm", "start"]
```

运行 **docker build .**

```shell
wuxianmimi docker-simpleweb % docker build .
[+] Building 18.2s (6/6) FINISHED
 => [internal] load build definition from Dockerfile                                                                              0.0s
 => => transferring dockerfile: 237B                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                 0.0s
 => => transferring context: 2B                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/node:14-alpine                                                                 3.5s
 => [1/2] FROM docker.io/library/node:14-alpine@sha256:80e825b1f5ab859498a2f0f98f8197131a562906e5f8c95977057502e68ca05a          13.0s
 => => resolve docker.io/library/node:14-alpine@sha256:80e825b1f5ab859498a2f0f98f8197131a562906e5f8c95977057502e68ca05a           0.0s
 => => sha256:f19b50748cdfe11c52ec028226d21a0cbc5a6b860e311b41a6299c2c43d1bfae 37.78MB / 37.78MB                                 12.1s
 => => sha256:7bfe7e7dc195d0082ab6b8c2bb09a8a405c51369b5c25c015327227f1390a312 2.43MB / 2.43MB                                    1.5s
 => => sha256:cb8b674ae1bc8e69210b956081a7c092daabfd2a4834f660ef3e0cadaab5db44 449B / 449B                                        1.9s
 => => sha256:80e825b1f5ab859498a2f0f98f8197131a562906e5f8c95977057502e68ca05a 1.43kB / 1.43kB                                    0.0s
 => => sha256:98f1a5d744f2343703913c3981098ac92fed74a6edec9133291d9f0ad4a03fe8 1.16kB / 1.16kB                                    0.0s
 => => sha256:8673fd44cb467741701c5006a1edca4f315bc1ac8af2964fe0a9586c7bbe195f 6.45kB / 6.45kB                                    0.0s
 => => extracting sha256:f19b50748cdfe11c52ec028226d21a0cbc5a6b860e311b41a6299c2c43d1bfae                                         0.7s
 => => extracting sha256:7bfe7e7dc195d0082ab6b8c2bb09a8a405c51369b5c25c015327227f1390a312                                         0.1s
 => => extracting sha256:cb8b674ae1bc8e69210b956081a7c092daabfd2a4834f660ef3e0cadaab5db44                                         0.0s
 => [2/2] RUN npm install                                                                                                         1.5s
 => exporting to image                                                                                                            0.0s
 => => exporting layers                                                                                                           0.0s
 => => writing image sha256:5a8723a245fdf92ef58101dd66e664c3365af9015b84adb30fc93a58dd25b07a                                      0.0s
```

构建成功，我们运行一下

```shell
wuxianmimi docker-simpleweb % docker run 5a8723a245fd
npm ERR! code ENOENT
npm ERR! syscall open
npm ERR! path /package.json
npm ERR! errno -2
npm ERR! enoent ENOENT: no such file or directory, open '/package.json'
npm ERR! enoent This is related to npm not being able to find a file.
npm ERR! enoent

npm ERR! A complete log of this run can be found in:
npm ERR!     /root/.npm/_logs/2023-01-09T02_53_07_209Z-debug.log
```

报错了，说是没有 package.json 这个文件，什么情况？（可能由于 Docker Engine 版本差异，视频中讲师在构建阶段就会报着个错误）

- **COPY 指令**

缺少 package.json 这个文件，是因为目前容器内的文件系统快照，是 node:14-alpine 这个镜像的拷贝，所以里面当然不会有我们的代码。

所以我们需要在构建镜像的时候，将我们的代码拷贝到容器内，这需要用到 **COPY** 指令：

```shell
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

**COPY** 指令把 <src> 的文件和文件夹，拷贝到容器内的文件系统 <dest> 中。

在我们 Dockerfile 中添加 **COPY** 指令：

```shell
# 三步曲1：规定基础镜像
FROM node:14-alpine

# 三步曲2：运行命令来安装必要的依赖、程序
COPY ./ ./
RUN npm install

# 三步曲3：规定容器启动参数
CMD ["npm", "start"]
```

这里咪咪建议在项目根目录创建一个 .npmrc 文件，在文件内添加一行：

```shell
registry=https://registry.npmmirror.com
```

这是 npm 镜像地址，对大陆用户比较友好，否则在容器内安装 npm 依赖的时间很长很长。。。（后面所有 NodeJS 项目都一样）

重新构建运行容器：

```shell
wuxianmimi docker-simpleweb % docker run 6c8d589ccc7e

> docker-simpleweb@1.0.0 start /
> node index.js

已监听8080端口
```

服务成功运行了，我们打开本地浏览器，在地址栏输入 localhost:8080 并回车：

![img]()

无法访问应用

咦，网站无法访问，什么情况？

- **端口映射**

当我们在本地浏览器前往 http://localhost:8080 时，请求的是本地 8080 端口，而不是容器内的 8080 端口。而我们本地并没有程序监听这个端口，所以也不会有响应。

默认情况下，容器内的端口是**不会\*\***接收\*\*到外部的请求的，我们需要在运行容器时，指定端口映射的规则。

在 **docker run** 命令添加 **-p** [本地端口]:[容器端口] 选项，增加端口映射：

```shell
wuxianmimi docker-simpleweb % docker run -p 8080:8080 b964820c57bd7

> docker-simpleweb@1.0.0 start /
> node index.js

已监听8080端口
```

重新在浏览器访问 localhost:8080，成功返回：

![img]()

获得返回数据

- **工作目录**

我们目前的项目文件，都是通过 **COPY** 指令拷贝到容器内的根目录的，这样的做法其实不太好。原因是根目录还有很多其他的文件，可能会与我们的项目有冲突。

```shell
/ # ls
Dockerfile         home               mnt                package.json       sbin               usr
bin                index.js           node_modules       proc               srv                var
dev                lib                opt                root               sys
etc                media              package-lock.json  run                tmp
```

解决办法就是在 Dockerfile 中使用 **WORKDIR** 指令，这个指令设置一个文件夹，成为之后 **RUN**、**CMD**、**COPY** 这些指令的起始目录。

更新我们的 Dockerfile：

```shell
# 三步曲1：规定基础镜像
FROM node:14-alpine

# 增加工作目录
WORKDIR /usr/app

# 三步曲2：运行命令来安装必要的依赖、程序
COPY ./ ./
RUN npm install

# 三步曲3：规定容器启动参数
CMD ["npm", "start"]
```

重新构建镜像并运行，现在在容器中的项目文件结构：

```shell
/usr/app # ls
Dockerfile         index.js           node_modules       package-lock.json  package.json
/usr/app # cd /
/ # ls
bin    dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var
/ #
```

- **优化构建**

目前构建镜像的流程已经走通了，但是仍有一个细节可以优化。

虽然 **docker build** 每一步指令都有缓存机制，只要上一步指令不更改，当前指令就会使用缓存，节省时间。但是如果我们现在更改了 index.js 的内容，我们肯定需要重新构建镜像，由于 **RUN** npm install 这步之前的指令肯定会变动（因为我们改了代码），所以每次更改代码后构建都需要重新安装依赖，比较浪费时间。

于是我们可以分两步 **COPY** 文件，先将 package.json 拷贝进容器，执行安装依赖的指令，再将其余文件拷贝进容器。这样的话，只要 package.json 不变，安装依赖这一步就可以用缓存的镜像。

```shell
# 三步曲1：规定基础镜像
FROM node:14-alpine

# 增加工作目录
WORKDIR /usr/app

# 三步曲2：运行命令来安装必要的依赖、程序
COPY ./package.json ./
COPY ./.npmrc ./
RUN npm install
COPY ./ ./

# 三步曲3：规定容器启动参数
CMD ["npm", "start"]
```

# 第五章：了解 Docker Compose

这一章要实现一个应用，当用户访问页面时，返回当前页面访问总次数。

- **准备项目**

新建文件夹，并在文件夹内创建：

package.json

```javascript
{
  "name": "docker-visists",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "*",
    "redis": "2.8.0"
  }
}
```

index.js

```javascript
const express = require("express");
const redis = require("redis");

const app = express();
const client = redis.createClient();
client.set("visits", 0);

app.get("/", (req, res) => {
  client.get("visits", (err, visits) => {
    res.send("访问次数 " + visits);
    client.set("visits", parseInt(visits) + 1);
  });
});

app.listen(8081, () => {
  console.log("监听8081端口中...");
});
```

Dockerfile

```shell
FROM node:14-alpine

WORKDIR '/app'

COPY package.json .
RUN npm install
COPY . .

CMD ["npm", "start"]
```

.npmrc（额外加的，提高 npm 依赖安装速度）

```shell
registry=https://registry.npmmirror.com
```

然后运行 **docker build** 命令打包镜像：

```shell
wuxianmimi docker-visists % docker build -t wuxianmimi/visits .
[+] Building 4.3s (11/11) FINISHED
 => [internal] load build definition from Dockerfile                                                                              0.0s
 => => transferring dockerfile: 159B                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                 0.0s
 => => transferring context: 2B                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/node:14-alpine                                                                 1.7s
 => [internal] load build context                                                                                                 0.0s
 => => transferring context: 507B                                                                                                 0.0s
 => [1/6] FROM docker.io/library/node:14-alpine@sha256:80e825b1f5ab859498a2f0f98f8197131a562906e5f8c95977057502e68ca05a           0.0s
 => CACHED [2/6] WORKDIR /app                                                                                                     0.0s
 => [3/6] COPY .npmrc .                                                                                                           0.0s
 => [4/6] COPY package.json .                                                                                                     0.0s
 => [5/6] RUN npm install                                                                                                         2.3s
 => [6/6] COPY . .                                                                                                                0.0s
 => exporting to image                                                                                                            0.1s
 => => exporting layers                                                                                                           0.1s
 => => writing image sha256:e5d5a301d20e68b4ebc59d451410ebbe9379bd2c3519acea6f1f3b6b12a9bfed                                      0.0s
 => => naming to docker.io/wuxianmimi/visits                                                                                      0.0s
```

- **Docker Compose**

Docker Compose 是 Docker 安装时一起安装的一个命令行工具，作用是使同时启动多个容器更方便。

我们通过 **docker-compose.yml** 文件来配置相关信息，在这个项目中，我们需要启动两个容器，一个是 redis，另一个是我们的 Node 应用。

docker-compose.yml

```yaml
# 使用的的 docker compose 文件标准
version: '3'
# 服务列表，一个容器可以理解为一个服务
services:
  # 容器1
  redis-server:
    # 使用的镜像
    image: 'redis'
  # 容器2
  node-app:
    # 构建当前项目
    build: .
    # 端口映射
    ports:
	  - "8081:8081"
```

services 列表中的容器，在网络层面是自动互通的，我们只需要在 Node 应用中，指明 redis 连接的 host 为 redis-server 即可，docker 会自动帮我们转发至对应名称的容器中。

- **启动**

通过命令 **docker-compose up** 来启动容器：

```shell
wuxianmimi docker-visists % docker-compose up
[+] Running 2/0
 ⠿ Container docker-visists-node-app-1      Created                                                                               0.0s
 ⠿ Container docker-visists-redis-server-1  Created                                                                               0.0s
Attaching to docker-visists-node-app-1, docker-visists-redis-server-1
docker-visists-redis-server-1  | 1:C 09 Jan 2023 13:17:30.111 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
docker-visists-redis-server-1  | 1:C 09 Jan 2023 13:17:30.111 # Redis version=7.0.7, bits=64, commit=00000000, modified=0, pid=1, just started
docker-visists-redis-server-1  | 1:C 09 Jan 2023 13:17:30.111 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
docker-visists-redis-server-1  | 1:M 09 Jan 2023 13:17:30.111 * monotonic clock: POSIX clock_gettime
docker-visists-redis-server-1  | 1:M 09 Jan 2023 13:17:30.112 * Running mode=standalone, port=6379.
docker-visists-redis-server-1  | 1:M 09 Jan 2023 13:17:30.112 # Server initialized
docker-visists-redis-server-1  | 1:M 09 Jan 2023 13:17:30.114 * Loading RDB produced by version 7.0.7
docker-visists-redis-server-1  | 1:M 09 Jan 2023 13:17:30.114 * RDB age 3 seconds
docker-visists-redis-server-1  | 1:M 09 Jan 2023 13:17:30.114 * RDB memory usage when created 0.92 Mb
docker-visists-redis-server-1  | 1:M 09 Jan 2023 13:17:30.114 * Done loading RDB, keys loaded: 1, keys expired: 0.
docker-visists-redis-server-1  | 1:M 09 Jan 2023 13:17:30.114 * DB loaded from disk: 0.000 seconds
docker-visists-redis-server-1  | 1:M 09 Jan 2023 13:17:30.114 * Ready to accept connections
docker-visists-node-app-1      |
docker-visists-node-app-1      | > docker-visists@1.0.0 start /app
docker-visists-node-app-1      | > node index.js
docker-visists-node-app-1      |
docker-visists-node-app-1      | 监听8081端口中...
```

浏览器访问 localhost:8081 页面：

![img]()

node-app 应用

需要停止时，通过 **docker-compose down** 来停止容器并移除容器与网络：

```shell
wuxianmimi docker-visists % docker-compose down
[+] Running 3/0
 ⠿ Container docker-visists-node-app-1      Removed                                                                               0.0s
 ⠿ Container docker-visists-redis-server-1  Removed                                                                               0.0s
 ⠿ Network docker-visists_default           Removed                                                                               0.1s
```

- **自动重启容器**

目前为止，如果我们的应用发生错误了，容器就停止了，那如何让容器关闭时自动重启呢？

可以在 docker-compose.yml 中配置 **restart** 属性：

```yaml
# 使用的的 docker compose 文件标准
version: "3"
# 服务列表，一个容器可以理解为一个服务
services:
  # 容器1
  redis-server:
    # 使用的镜像
    image: "redis"
  # 容器2
  node-app:
    # 程序永远会在停止时自动重启
    restart: always
    # 构建当前项目
    build: .
    # 端口映射
    ports:
      - "8081:8081"
```

restart 一共有四种策略

![img]()

restart 策略

- **docker-compose ps**

与 **docker ps** 类似，**docker-compose ps** 可以查看容器的状态（不只是正在运行的）：

```shell
wuxianmimi docker-visists % docker-compose ps
NAME                            COMMAND                  SERVICE             STATUS              PORTS
docker-visists-node-app-1       "docker-entrypoint.s…"   node-app            exited (0)
docker-visists-redis-server-1   "docker-entrypoint.s…"   redis-server        exited (0)
```

⚠️：与 docker 命令不同，docker-compose 命令，都需要目录内有 docker-compose.yml 文件，否则会报错。

# 第六章：构建自定义镜像+（Nginx + React）

接下来两章讲解 Docker 开发的工作流，了解 Docker 在开发、测试、部署、开发循环流程中扮演的角色：

开发：介绍了如何使用 Github 管理代码

测试：使用 Travis CI 进行持续集成

部署：在亚马逊 AWS 上部署

- **初始化**

使用 **npx create-react-app docker-react-frontend** 初始化前端项目。（过程可能有点长，可增加选项跳过安装步骤）

添加 .npmrc 文件。

添加 Dockerfile.dev 文件，开发环境的构建镜像配置：

```shell
FROM node:14-alpine

WORKDIR '/usr/app'

COPY .npmrc .
COPY package.json .
RUN npm install
COPY . .

CMD ["npm", "run", "start"]
```

这一章区分开发环境和生产环境，项目中 Dockerfile 是区分的，使用 Dockerfile.dev 配置开发环境，构建时添加 -f 选项指明配置文件：docker build -f Dockerfile.dev .

- **Volumes**

视频中在前端项目时，如果我们改动代码，不会影响容器中的代码，容器内使用还是构建时的代码，而应用又是在容器内启动的，所以也不会有热更新。

而我们不可能改一次代码就构建一次镜像，那样的话太麻烦了，还有没有其他什么方法？

答案就是要用 Docker 中 Volumes 这个机制。

Volumes 可能理解起来比较抽象，可以先简单的等同端口映射，我们将本地的文件“映射”到容器内部。

实际做法是 docker run 命令增加 -v 选项：

```shell
wuxianmimi docker-react-frontend % docker run -p 3000:3000 -v /usr/app/node_modules -v $(pwd):/usr/app 72f579b2bf49e403

> docker-react-frontend@0.1.0 start /usr/app
> react-scripts start

(node:25) [DEP_WEBPACK_DEV_SERVER_ON_AFTER_SETUP_MIDDLEWARE] DeprecationWarning: 'onAfterSetupMiddleware' option is deprecated. Please use the 'setupMiddlewares' option.
(Use `node --trace-deprecation ...` to show where the warning was created)
(node:25) [DEP_WEBPACK_DEV_SERVER_ON_BEFORE_SETUP_MIDDLEWARE] DeprecationWarning: 'onBeforeSetupMiddleware' option is deprecated. Please use the 'setupMiddlewares' option.
Starting the development server...

Compiled successfully!

You can now view docker-react-frontend in the browser.

  Local:            http://localhost:3000
  On Your Network:  http://172.17.0.2:3000

Note that the development build is not optimized.
To create a production build, use npm run build.

webpack compiled successfully
Compiling...
Compiled successfully!
webpack compiled successfully
```

第一个 **-v /usr/app/node_modules**，没有冒号， 表示容器内这个文件夹不需要映射，使用容器本身的文件。

第二个 **-v $(pwd):/usr/app**，在有冒号的情况下，左边的映射到右边。冒号左边表示本地目录，冒号右边表示容器内目录。

以上操作也可以使用 docker-compose 来简化，添加 docker-compose.yml 文件：

```yaml
version: "3"
services:
  web:
    # 这里同之前写法不一样，因为默认是找 Dockerfile 文件的
    # 由于我们的文件名不一样，所以要手动配置
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /usr/app/node_modules
      - .:/usr/app
```

运行 **docker-compose up** 命令，顺利启动。

- **测试**

docker-compose.yml 增加第二个服务，用来测试应用：

```yaml
version: "3"
services:
  web:
    # 这里同之前写法不一样，因为默认是找 Dockerfile 文件的
    # 由于我们的文件名不一样，所以要手动配置
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /usr/app/node_modules
      - .:/usr/app
  tests:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /usr/app/node_modules
      - .:/usr/app
    # 覆盖默认命令
    command: ["npm", "run", "test"]
```

运行 **docker-compose up** 命令时，会自动运行测试服务。

- **使用 Nginx 来做 web 服务器**

前端应用使用 npm run build 打包成静态文件后放在服务器上，然后请求通过 Nginx 访问这些文件。

这一步的难点有两个，第一，打包需要安装依赖，而依赖又很大占空间怎么办？第二，打包的文件和 Ngnix 如何结合？

创建 Dockerfile 文件：

```shell
# 给第一步构建阶段取别名 builder
FROM node:14-alpine as builder
WORKDIR '/usr/app'
COPY .npmrc .
COPY package.json .
RUN npm install
COPY . .
RUN ["npm", "run", "build"]

# FROM 指令划分构建阶段，下面是第二步
FROM nginx
# 指明文件从 builder 阶段产生的镜像中来
COPY --from=builder /usr/app/build /usr/share/nginx/html
# nginx 镜像有默认启动命令，不需要我们设置
```

命令行输入 **docker build .** 构建，构建成功后输入 **docker run** 命令启动容器：

```shell
wuxianmimi docker-react-frontend % docker run -p 80:80 2d16f24ae5fbeefe00
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/01/10 07:03:18 [notice] 1#1: using the "epoll" event method
2023/01/10 07:03:18 [notice] 1#1: nginx/1.23.3
2023/01/10 07:03:18 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2023/01/10 07:03:18 [notice] 1#1: OS: Linux 5.15.49-linuxkit
2023/01/10 07:03:18 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/01/10 07:03:18 [notice] 1#1: start worker processes
2023/01/10 07:03:18 [notice] 1#1: start worker process 29
2023/01/10 07:03:18 [notice] 1#1: start worker process 30
2023/01/10 07:03:18 [notice] 1#1: start worker process 31
2023/01/10 07:03:18 [notice] 1#1: start worker process 32
```

浏览器访问 http://localhost/

![img]()

React 项目

#

# 第七章：持续集成与部署（Travis CI + AWS Elastic Beanstalk）

这一章紧接上一章节，真正开始在 Github、Travis CI、AWS 上进行操作。

略。

# 第八章：多容器应用：准备阶段（Nginx + React + Express + Postgres + Redis）

接下来将继续提高应用架构复杂度，更多容器情况下的开发流程。

容器 1：前端 react 客户端

容器 2：express web 服务器

容器 3：计算斐波那契数列的 worker

容器 4：Nginx

容器 5：redis

容器 6：Postgres

这个项目实现了以下效果，用户请求页面后，在页面可以输入一个数字发送给服务端，服务端响应请求并在 worker 中计算用户发来数字在斐波那契数列对应项的值。

页面上有存储在 Postgres 的历史输入值，还有存储在 redis 的计算结果。

这一章节都是在实现容器 1、容器 2、容器 3 这三个应用，如果只想关注在 Docker 技术的可略过。

这一章节完成后的代码模板，懒得自己敲的话可以用咪咪的：

https://github.com/lyf61/Docker-and-Kubernetes-The-Complete-Guide-Complex-App-Starter

# 第九章：多容器应用：与 Docker 结合

经过上一章后，我们项目目前文件结构如下：

![img]()

代码项目结构

client 是 React 项目，server 是 Express 服务器，worker 是计算斐波那契数列的程序。

课程先从 client 开始，在 client 目录中创建 Dockerfile.dev：

```shell
FROM node:14-alpine

WORKDIR '/app'

# 不要忘了创建 .npmrc 来节省安装依赖时间
COPY ./.npmrc .
COPY ./package.json .
RUN npm install
COPY . .

CMD ["npm", "run", "start"]
```

server 与 worker 都是 NodeJS 项目，所以创建的 Dockerfile.dev 一样：

```shell
# 这里用Node12版本
# 原因是连接 postgres 数据库的 pg@7.4.3（视频中的版本）不支持Node14
# 否则会有BUG
FROM node:12-alpine

WORKDIR '/app'

COPY ./.npmrc .
COPY ./package.json .
RUN npm install
COPY . .

# 与 client 唯一的不同
CMD ["npm", "run", "dev"]
```

然后在整个项目根目录增加 docker-compose.yaml：

```shell
version: '3'
services:
  # postgres数据库
  postgres:
    image: 'postgres:10.5'
  # redis数据库
  redis:
    image: 'redis:4-alpine'
  # Express 服务器
  api:
    build:
      dockerfile: Dockerfile.dev
      context: ./server
    volumes:
      - /app/node_modules
      - ./server:/app
    # 通过 environment 设置容器的**运行时**环境变量
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - PGUSER=postgres
      - PGHOST=postgres
      - PGDATABASE=postgres
      - PGPASSWORD=postgres_password
      - PGPORT=5432
  # React
  client:
    build:
      dockerfile: Dockerfile.dev
      context: ./client
    volumes:
      - /app/node_modules
      - ./client:/app
  # 计算斐波那契数列程序
  worker:
    build:
      dockerfile: Dockerfile.dev
      context: ./worker
    volumes:
      - /app/node_modules
      - ./worker:/app
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
```

相比之前的配置，这里新增了 **environment**，可以配置容器**运行时**的环境变量。

- **Nginx**

根据一开始的架构设计，最终暴露出来的服务是 Nginx，请求经过 Nginx 处理后，转发至 React 侧或 Express 侧。

根目录新增 nginx 文件夹来存放 Nginx 项目配置，并在目录内增加两个文件：

default.conf

```nginx
upstream client {
    server client:3000;
}

upstream api {
    server api:5000;
}

server {
    listen 80;

    location / {
        proxy_pass http://client;
    }

		location /sockjs-node {
        proxy_pass http://client;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }

    location /api {
        rewrite /api/(.*) /$1 break;
        proxy_pass http://api;
    }
}
```

Dockerfile.dev

```shell
FROM nginx:1.15.2-alpine

COPY ./default.conf /etc/nginx/conf.d/default.conf
```

同时在根目录的 docker-compose.yaml 中新增 Nginx 的配置，完整配置如下：

```yaml
version: "3"
services:
  # postgres数据库
  postgres:
    image: "postgres:10.5"
  # redis数据库
  redis:
    image: "redis:4-alpine"
  # nginx
  ngxin:
    restart: always
    build:
      dockerfile: Dockerfile.dev
      context: ./nginx
    ports:
      - "3030:80"
  # Express 服务器
  api:
    build:
      dockerfile: Dockerfile.dev
      context: ./server
    volumes:
      - /app/node_modules
      - ./server:/app
    # 通过 environment 设置容器的**运行时**环境变量
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - PGUSER=postgres
      - PGHOST=postgres
      - PGDATABASE=postgres
      - PGPASSWORD=postgres_password
      - PGPORT=5432
  # React
  client:
    build:
      dockerfile: Dockerfile.dev
      context: ./client
    volumes:
      - /app/node_modules
      - ./client:/app
  # 计算斐波那契数列程序
  worker:
    build:
      dockerfile: Dockerfile.dev
      context: ./worker
    volumes:
      - /app/node_modules
      - ./worker:/app
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
```

命令行输入 **docker-compose up —build** 来启动项目，在浏览器输入 http://localhost:3030/ 访问特面，跳出：

![img]()

启动项目成功

# 第十章：多容器应用：持续集成

略。

# 第十一章：多容器应用：部署至 AWS

略。

# 第十二章：Kubernetes 介绍与安装

从这一章节开始正式进入 Kubernetes 的世界。

- **Kubernetes 是什么？**

一套在多个机器上运行多个容器的系统。

- **为什么用 Kubernetes？**

在容器很多时，使得管理容器和镜像更简单。

- 一些概念

1. Nodes

   节点，一个虚拟机或者物理机器。

2. Pod

   一组容器，运行在节点中。Kubernetes 中管理的**最小计算单元**。

3. Kubernetes Clusters

   Kubernetes 集群，可以理解为 Master + Nodes。

4. Master

   负责管理集群的一个虚拟机或者物理机器。

5. minikube

   让我们在本地运行 Kubernetes 的命令行工具。

6. kubectl

   与 Master 交互的命令行工具。

- **安装**

现在安装 Docker 会自动安装上 kubectl，所以我们只要安装 minikube 就可以了。

minikube 官方安装文档：https://minikube.sigs.k8s.io/docs/start/

选择相应的版本安装完毕后即可。

安装完成后，使用 **minikube start** 命令来启动集群。

启动需要花点时间安装，启动后，可以通过 **minikube status** 和 **kubectl cluster-info** 这两个命令查看集群运行信息：

```shell
wuxianmimi docker-complex % minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

**kubectl cluster-info**

```shell
wuxianmimi docker-complex % kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:63259
CoreDNS is running at https://127.0.0.1:63259/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

创建文件夹 kubernetes-simplek8s，并在文件夹内创建 2 个文件：

client-pod.yaml

```yaml
# Kubernetes API 的版本
apiVersion: v1
# 这个配置文件生成的对象类型
# Pod 就是一个或一组容器
kind: Pod
# 唯一标识对象的一些数据
metadata:
  name: client-pod
  labels:
    component: web
# 对象的状态
spec:
  containers:
	# 容器名，任意取
    - name: client
	# 使用的镜像，镜像必须发布在 Docker Hub 上才能用
      image: stephengrider/multi-client
	# 暴露的端口
      ports:
        - containerPort: 3000
```

client-node-port.yaml

```yaml
apiVersion: v1
# Service 类型，设置集群中的网络
kind: Service
metadata:
  name: client-node-port
spec:
	# 子类型，一共有四种取值：
	# 1.ClusterIP，默认值，暴露内部IP，使得集群内部其他对象可以访问
	# 2.NodePort，暴露内部IP+端口，仅用于开发目的
	# 3.LoadBalancer，云提供商的负载均衡器向外部暴露服务
	# 4.ExternalName，将服务映射到 DNS 名称
  type: NodePort
  ports:
    - port: 3050
      targetPort: 3000
	# 默认：30000-32767
      nodePort: 31515
  selector:
    component: web
```

当我们从外部请求 client 时，请求通过 kube-proxy 发送到 Service，Service 再转发到 Pod 暴露的 3000 端口。

配置文件创建完成后，我们使用 **kubectl apply** 命令对资源应用配置更改：

```shell
kubectl apply -f FILENAME [flags]
```

运行命令，**-f** 选项传入对应配置文件：

```shell
wuxianmimi kubernetes-simplek8s % kubectl apply -f ./client-pod.yaml
pod/client-pod created
wuxianmimi kubernetes-simplek8s % kubectl apply -f ./client-node-port.yaml
service/client-node-port created
wuxianmimi kubernetes-simplek8s %
```

随后使用 **kubectl get pods** 查看正在运行的 Pods 列表：

```shell
kubectl get (-f FILENAME | TYPE [NAME | /NAME | -l label]) [--watch] [--sort-by=FIELD] [[-o | --output]=OUTPUT_FORMAT] [flags]
```

获取 Pod 列表：

```shell
wuxianmimi kubernetes-simplek8s % kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
client-pod   1/1     Running   0          2m37s
wuxianmimi kubernetes-simplek8s %
```

使用 **kubectl get services** 查看正在运行的服务：

```shell
wuxianmimi kubernetes-simplek8s % kubectl get services
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
client-node-port   NodePort    10.102.151.149   <none>        3050:31515/TCP   3m6s
kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP          150m
wuxianmimi kubernetes-simplek8s %
```

接下来就是准备访问服务了，需要访问 minikube 中的服务，只能通过 ip 来访问，使用 **minikube ip** 获取 ip 地址：

```shell
wuxianmimi kubernetes-simplek8s % minikube ip
192.168.49.2
wuxianmimi kubernetes-simplek8s %
```

访问 http://192.168.49.2:31515/ 可以看到 multi-client 这个页面。

⚠️：这里就会碰到咪咪在文章开头说到的问题，如果你使用 Mac 的话，由于 Docker Desktop 的实现问题，无法使用 IP 地址访问容器。可以在命令行输入 minikube service client-node-port --url 来获取一个本机地址，通过该地址来访问页面。

![img]()

Kubernetes 启动应用成功！

# 第十三章：了解 Kubernetes 配置

- **更新镜像**

在 client-pod.yaml 中，我们更改 image 为 **stephengrider/multi-worker**：

```yaml
# Kubernetes API 的版本
apiVersion: v1
# 这个配置文件生成的对象类型
kind: Pod
# 唯一标识对象的一些数据
metadata:
  name: client-pod
  labels:
    component: web
# 对象的状态
spec:
  containers:
    # 容器名，任意取
    - name: client
      # 使用的镜像，镜像必须发布在 Docker Hub 上才能用
      image: stephengrider/multi-worker
      # 暴露的端口
      ports:
        - containerPort: 3000
```

随后在命令行使用 **kubectl apply** 命令更新配置：

```yaml
wuxianmimi kubernetes-simplek8s % kubectl apply -f ./client-pod.yaml
pod/client-pod configured
```

过一会儿使用 **kubectl get pods** 查看 pod 状态，发现已经重启了：

```yaml
wuxianmimi kubernetes-simplek8s % kubectl get pods
NAME         READY   STATUS    RESTARTS      AGE
client-pod   1/1     Running   1 (58s ago)   19h
```

如果需要详细的资源情况，可以使用 **kubectl describe** 命令来获取：

```shell
kubectl describe (-f FILENAME | TYPE [NAME_PREFIX | /NAME | -l label]) [flags]
```

获取 client-pod 详情：

```shell
wuxianmimi kubernetes-simplek8s % kubectl describe pod client-pod
Name:             client-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Thu, 12 Jan 2023 16:33:36 +0800
Labels:           component=web
Annotations:      <none>
Status:           Running
IP:               172.17.0.3
IPs:
  IP:  172.17.0.3
Containers:
  client:
    Container ID:   docker://e99f30ff3b6fc654849e44e79c2ee74c5cc084271b1d4f66c9443e2a715bce0e
    Image:          stephengrider/multi-worker
    Image ID:       docker-pullable://stephengrider/multi-worker@sha256:5fbab5f86e6a4d499926349a5f0ec032c42e7f7450acc98b053791df26dc4d2b
    Port:           3000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 13 Jan 2023 11:33:44 +0800
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Thu, 12 Jan 2023 16:34:00 +0800
      Finished:     Fri, 13 Jan 2023 11:33:00 +0800
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7797r (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-7797r:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason   Age                  From     Message
  ----    ------   ----                 ----     -------
  Normal  Killing  4m8s                 kubelet  Container client definition changed, will be restarted
  Normal  Pulling  4m8s                 kubelet  Pulling image "stephengrider/multi-worker"
  Normal  Created  3m24s (x2 over 19h)  kubelet  Created container client
  Normal  Started  3m24s (x2 over 19h)  kubelet  Started container client
  Normal  Pulled   3m24s                kubelet  Successfully pulled image "stephengrider/multi-worker" in 43.695453104s
wuxianmimi kubernetes-simplek8s %
```

输出的信息尾部可以看到有拉取"**stephengrider/multi-worker**"镜像的记录。

- **Deployment**

刚刚我们更新 image 成功了，但是如果使用这种方法更新端口，你会发现无法更新，在 **kubectl apply** 时会报错。

当对象配置 kind 为 Pod 时，我们只能更新部分字段，其中不包括端口，而如果要更全面的声明式更新能力，我们需要使用 Deployment。

在文件夹内创建 client-deployment.yaml：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-deployment
spec:
  # Pod 数量
  replicas: 1
  # Pod 选择器
  selector:
    matchLabels:
      component: web
  template:
    # Pod
    metadata:
      labels:
        component: web
    spec:
      containers:
        - name: client
          image: stephengrider/multi-client
          ports:
            - containerPort: 3000
```

完成配之后，我们先删除之前创建的 Pod，使用命令 **kubectl delete** 来完成删除工作：

```shell
kubectl delete (-f FILENAME | TYPE [NAME | /NAME | -l label | --all]) [flags]
```

删除 client-pod.yaml 的相关配置：

```shell
wuxianmimi kubernetes-simplek8s % kubectl delete -f ./client-pod.yaml
pod "client-pod" deleted
wuxianmimi kubernetes-simplek8s % kubectl get pods
No resources found in default namespace.
wuxianmimi kubernetes-simplek8s %
```

删除后，我们应用新的配置：

```shell
wuxianmimi kubernetes-simplek8s % kubectl apply -f ./client-deployment.yaml
deployment.apps/client-deployment created
wuxianmimi kubernetes-simplek8s % kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
client-deployment   1/1     1            1           21s
wuxianmimi kubernetes-simplek8s %
```

接下来更改 client-deployment.yaml 中 containerPort 后，再应用配置就不会报错了。

- **更新镜像版本**

如果要更新镜像版本，可以使用：

**kubectl set image [object 类型]/[object name] [容器 name]=[新镜像版本]**

- **切换 Docker-client 请求指向**

使用 **eval $(minikube docker-env)** 命令，可以将 docker-cli 指向的 docker-server 变成 minikube 创建的虚拟机中，可以方便我们调试：

```shell
wuxianmimi kubernetes-simplek8s % eval $(minikube docker-env)
wuxianmimi kubernetes-simplek8s % docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS     NAMES
bff39aba4d5b   6d3ffc2696ac           "/coredns -conf /etc…"   8 minutes ago   Up 8 minutes             k8s_coredns_coredns-78fcd69978-x84sp_kube-system_b16c591f-cd49-4967-9489-f4654834f8aa_0
5f2dc9090d50   d9fa9053808e           "/usr/local/bin/kube…"   8 minutes ago   Up 8 minutes             k8s_kube-proxy_kube-proxy-9bdkr_kube-system_c8cae802-b82a-455b-a44f-791f1541bb0b_0
f591feca3db1   k8s.gcr.io/pause:3.5   "/pause"                 8 minutes ago   Up 8 minutes             k8s_POD_coredns-78fcd69978-x84sp_kube-system_b16c591f-cd49-4967-9489-f4654834f8aa_0
3d4d77d0ecbc   k8s.gcr.io/pause:3.5   "/pause"                 8 minutes ago   Up 8 minutes             k8s_POD_kube-proxy-9bdkr_kube-system_c8cae802-b82a-455b-a44f-791f1541bb0b_0
428446d5c4cc   ba04bb24b957           "/storage-provisioner"   8 minutes ago   Up 8 minutes             k8s_storage-provisioner_storage-provisioner_kube-system_785bb891-8438-45f8-95ea-2fb7511af55a_0
5611755fac61   k8s.gcr.io/pause:3.5   "/pause"                 8 minutes ago   Up 8 minutes             k8s_POD_storage-provisioner_kube-system_785bb891-8438-45f8-95ea-2fb7511af55a_0
fe491e1de92c   4641e56315a2           "kube-scheduler --au…"   9 minutes ago   Up 9 minutes             k8s_kube-scheduler_kube-scheduler-minikube_kube-system_6fd078a966e479e33d7689b1955afaa5_0
87aac0b31bd5   d5504eacf2d7           "kube-controller-man…"   9 minutes ago   Up 9 minutes             k8s_kube-controller-manager_kube-controller-manager-minikube_kube-system_f8d2ab48618562b3a50d40a37281e35e_0
86681a2dabf7   7605412e3e07           "kube-apiserver --ad…"   9 minutes ago   Up 9 minutes             k8s_kube-apiserver_kube-apiserver-minikube_kube-system_cf344d9adbf2feb59a7c63da2c6283f8_0
1ae60d767e22   2252d5eb703b           "etcd --advertise-cl…"   9 minutes ago   Up 9 minutes             k8s_etcd_etcd-minikube_kube-system_cc51499f38934c109ed290475d9492d0_0
ed08b7a7550c   k8s.gcr.io/pause:3.5   "/pause"                 9 minutes ago   Up 9 minutes             k8s_POD_kube-scheduler-minikube_kube-system_6fd078a966e479e33d7689b1955afaa5_0
ce7d76aff58a   k8s.gcr.io/pause:3.5   "/pause"                 9 minutes ago   Up 9 minutes             k8s_POD_kube-controller-manager-minikube_kube-system_f8d2ab48618562b3a50d40a37281e35e_0
9f42fd98a0f2   k8s.gcr.io/pause:3.5   "/pause"                 9 minutes ago   Up 9 minutes             k8s_POD_kube-apiserver-minikube_kube-system_cf344d9adbf2feb59a7c63da2c6283f8_0
d17349c2242c   k8s.gcr.io/pause:3.5   "/pause"                 9 minutes ago   Up 9 minutes             k8s_POD_etcd-minikube_kube-system_cc51499f38934c109ed290475d9492d0_0
wuxianmimi kubernetes-simplek8s %
```

# 第十四章：多容器应用：Kubernetes 开发环境搭建

这一章开始使用 Kubernetes 部署更复杂的项目，使用的是学习 docker-compose 时的 docker-complex 那个应用，Kubernetes 版本的架构如下图：

![img]()

kubernetes-complex 项目架构

一共需要 11 个配置文件。

启动模板可以使用咪咪的：

https://github.com/lyf61/Docker-and-Kubernetes-The-Complete-Guide-Complex-App-Kubernetes-Starter

不过由于我们不会部署在生产环境，且我们使用的 Docker 镜像都是讲师已经发布的，所以开发环境下完全不使用模板也可以。

首先创建 client 的 service 与 deployment，在项目根目录下创建 k8s 文件夹，随后创建两个文件：

client-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      component: web
  template:
    metadata:
      labels:
        component: web
    spec:
      containers:
        - name: client
          image: stephengrider/multi-client
          ports:
            - containerPort: 3000
```

client-cluster-ip-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: client-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: web
  ports:
    - port: 3000
      targetPort: 3000
```

随后使用 **kubectl apply** 应用配置测试一下，这次 **-f** 选项传入文件夹，kubectl 会自动应用文件夹下所有配置：

```shell
wuxianmimi kubernetes-complex % kubectl apply -f k8s
service/client-cluster-ip-service created
deployment.apps/client-deployment created
wuxianmimi kubernetes-complex % kubectl get deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
client-deployment   3/3     3            3           66s
wuxianmimi kubernetes-complex % kubectl get pod
NAME                                 READY   STATUS    RESTARTS   AGE
client-deployment-7cb6c958f7-4c4pc   1/1     Running   0          72s
client-deployment-7cb6c958f7-4kp2x   1/1     Running   0          72s
client-deployment-7cb6c958f7-gv78d   1/1     Running   0          72s
wuxianmimi kubernetes-complex % kubectl get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
client-cluster-ip-service   ClusterIP   10.109.207.213   <none>        3000/TCP   80s
kubernetes                  ClusterIP   10.96.0.1        <none>        443/TCP    18h
wuxianmimi kubernetes-complex %
```

可以看到启动成功了。

继续创建 server 与 worker 相关对象配置：

server-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: server-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      component: server
  template:
    metadata:
      labels:
        component: server
    spec:
      containers:
        - name: server
          image: stephengrider/multi-server
          ports:
            - containerPort: 5000
```

server-cluster-ip-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: server-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: server
  ports:
    - port: 5000
      targetPort: 5000
```

worker-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      component: worker
  template:
    metadata:
      labels:
        component: worker
    spec:
      containers:
        - name: worker
          image: stephengrider/multi-worker
```

测试后，server Pod 中的日志应该显示数据库连接不上，说明其他配置没问题：

```shell
wuxianmimi kubernetes-complex % kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
client-deployment-7cb6c958f7-4c4pc   1/1     Running   0          32m
client-deployment-7cb6c958f7-4kp2x   1/1     Running   0          32m
client-deployment-7cb6c958f7-gv78d   1/1     Running   0          32m
server-deployment-9bff8dfb-cc7ls     1/1     Running   0          3m13s
server-deployment-9bff8dfb-dktx4     1/1     Running   0          3m13s
server-deployment-9bff8dfb-m9mg8     1/1     Running   0          3m13s
worker-deployment-666c96ffc5-n6gh8   1/1     Running   0          3m13s
wuxianmimi kubernetes-complex % kubectl logs server-deployment-9bff8dfb-m9mg8

> @ start /app
> node index.js

Listening
{ Error: connect ECONNREFUSED 127.0.0.1:5432
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1161:14)
  errno: 'ECONNREFUSED',
  code: 'ECONNREFUSED',
  syscall: 'connect',
  address: '127.0.0.1',
  port: 5432 }
```

接下来配置 Redis 与 Postgres：

redis-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: redis
  template:
    metadata:
      labels:
        component: redis
    spec:
      containers:
        - name: redis
          image: redis:4-alpine
          ports:
            - containerPort: 6379
```

redis-cluster-ip-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: redis
  ports:
    - port: 6379
      targetPort: 6379
```

postgres-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: postgres
  template:
    metadata:
      labels:
        component: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:10.5
          ports:
            - containerPort: 5432
```

postgres-cluster-ip-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: postgres
  ports:
    - port: 5432
      targetPort: 5432
```

- **Kubernetes 中的 Volume**

在本项目架构图中，我们发现一个叫 **Postgres PVC** 的东东。如果没有这个对象，当我们 postgres 容器被销毁的时候，postgres 中的数据也同时被销毁了。所以这个对象有点类似我们之前学习 Docker 时 Volume 的概念，但是 Docker 中 Volume 是一种让容器访问外部文件系统的机制，而 Kubernetes 中的 Volume 是一个对象，在 Pod 层面存储数据。

![img]()

Postgres PVC

- **PersistentVolume**

Kubernetes 中的 Volume 在 Pod 层面存储数据，所以 Pod 的生命周期也会影响数据，这不太好。而 PersistentVolume 解决了这个问题，把数据从 Pod 中“剥离”了出来。

- **PersistentVolumeClaim**

乍一眼看上去与 PersistentVolume 很像，这两个有什么区别？在[课程的 P186 视频](https://www.bilibili.com/video/BV1hS4y1m7Ma/?p=186)讲师用火柴人动画讲解了很清楚，可以看一下。简单的来说就是 PersistentVolume 是预先静态分配好的一个存储资源，而 PersistentVolumeClaim 是对存储空间的请求和申领，等需要时才会提供。

创建我们的 PersistentVolumeClaim，在 k8s 目录创建文件 database-persistent-volume-cliam.yaml：

```yaml
apiVersion: v1
# 对于我们来说新的对象类型
kind: PersistentVolumeClaim
metadata:
  name: database-persistent-volume-claim
spec:
  # 访问模式，一共有四种取值：
  # ReadWriteOnce    卷可以被 一个节点 以读写方式挂载
  # ReadOnlyMany     卷可以被 多个节点 只读方式挂载
  # ReadWriteMany    卷可以被 多个节点 以读写方式挂载
  # ReadWriteOncePod 卷可以被 单个Pod  以读写方式挂载
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      # 要求2GB存储空间
      storage: 2Gi
```

- **StorageClass**

存储 "类" 的方法，简单来说就是申请一块存储空间时的策略和方法的提供者。

在本地环境中只有一种方式，在云服务上有各个云服务器厂商提供的“类”，可以通过命令行 **kubectl get storageclass** 来查看当前支持哪些 StorageClass：

```shell
wuxianmimi kubernetes-complex % kubectl get storageclass
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  20h
wuxianmimi kubernetes-complex %
```

随后在 postgres-deployment.yaml 中增加 Volume 的相关配置：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: postgres
  template:
    metadata:
      labels:
        component: postgres
    spec:
      # 可以由属于 Pod 的容器挂载的卷列表
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: database-persistent-volume-claim
      containers:
        - name: postgres
          image: postgres:10.5
          ports:
            - containerPort: 5432
          # 要挂载到容器文件系统中的 Pod 卷
          volumeMounts:
            - name: postgres-storage
              # 在容器内 Volume 的挂载路径
              mountPath: /var/lib/postgresql/data
              # Volume 中的路径，默认为 ""（卷的根）。
              subPath: postgres
```

应用配置后，我们在命令行查看 pv 与 pvc 的情况：

```shell
wuxianmimi kubernetes-complex % kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                      STORAGECLASS   REASON   AGE
pvc-a62cbd16-f788-4daf-b6e2-e3ec77ebdc32   2Gi        RWO            Delete           Bound    default/database-persistent-volume-claim   standard                28s
wuxianmimi kubernetes-complex % kubectl get pvc
NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
database-persistent-volume-claim   Bound    pvc-a62cbd16-f788-4daf-b6e2-e3ec77ebdc32   2Gi        RWO            standard       30s
wuxianmimi kubernetes-complex %
```

- **环境变量**

到目前为止我们还没有设置我们的环境变量，那如何设置数据库的连接信息呢？

在 worker-deployment.yaml 中增加 Redis 的环境变量：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: worker
  template:
    metadata:
      labels:
        component: worker
    spec:
      containers:
        - name: worker
          image: stephengrider/multi-worker
          # 设置环境变量
          env:
            - name: REDIS_HOST
              value: redis-cluster-ip-service
            - name: REDIS_PORT
              value: "6379"
```

在 server-deployment.yaml 中，增加 Redis 和 Postgres 的环境变量：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: server-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      component: server
  template:
    metadata:
      labels:
        component: server
    spec:
      containers:
        - name: server
          image: stephengrider/multi-server
          ports:
            - containerPort: 5000
          # 设置环境变量
          env:
            - name: REDIS_HOST
              value: redis-cluster-ip-service
            - name: REDIS_PORT
              value: "6379"
            - name: PGUSER
              value: postgres
            - name: PGHOST
              value: postgres-cluster-ip-service
            - name: PGPORT
              value: "5432"
            - name: PGDATABASE
              value: postgres
```

- **Secret**

还剩最后的 Postgres 密码没有配置，这种敏感信息用明文写在配置文件中不太合适，Kubernetes 为我们提供了一种方式来存储这些敏感信息，这个对象叫 Secret。

使用 **kubectl create secret generic pgpassword --from-literal PGPASSWORD=12345asdf** 命令创建 Secret。

获取目前 sercet：

```shell
wuxianmimi kubernetes-complex % kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-mdzch   kubernetes.io/service-account-token   3      26h
pgpassword            Opaque                                1      22s
wuxianmimi kubernetes-complex %
```

sercet 是存在环境变量中的，不同的机器上部署要重新设置 secret。

随后在 server 与 postgres 中添加密码（**secretKeyRef**）：

server-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: server-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      component: server
  template:
    metadata:
      labels:
        component: server
    spec:
      containers:
        - name: server
          image: stephengrider/multi-server
          ports:
            - containerPort: 5000
          # 设置环境变量
          env:
            - name: REDIS_HOST
              value: redis-cluster-ip-service
            - name: REDIS_PORT
              value: "6379"
            - name: PGUSER
              value: postgres
            - name: PGHOST
              value: postgres-cluster-ip-service
            - name: PGPORT
              value: "5432"
            - name: PGDATABASE
              value: postgres
            - name: PGPASSWORD
              # 值来自我们刚刚设置的 Secret
              valueFrom:
                secretKeyRef:
                  name: pgpassword
                  key: PGPASSWORD
```

postgres-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: postgres
  template:
    metadata:
      labels:
        component: postgres
    spec:
      # 可以由属于 Pod 的容器挂载的卷列表
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: database-persistent-volume-claim
      containers:
        - name: postgres
          image: postgres:10.5
          ports:
            - containerPort: 5432
          # 要挂载到容器文件系统中的 Pod 卷
          volumeMounts:
            - name: postgres-storage
              # 在容器内 Volume 的挂载路径
              mountPath: /var/lib/postgresql/data
              # Volume 中的路径，默认为 ""（卷的根）。
              subPath: postgres
          env:
            - name: PGPASSWORD
              # 值来自我们刚刚设置的 Secret
              valueFrom:
                secretKeyRef:
                  name: pgpassword
                  key: PGPASSWORD
```

# 第十五章：了解 Ingress 与 Ingress Nginx 配置

在 Kubernetes 中，Ingress 暴露出 HTTP 和 HTTPS 路由，可以接受外部流量后转发内部服务，相当于一个入口。

课程中使用的 Ingress 实现是 Nginx Ingress，使用的是 ingress-nginx 这个由社区维护的项目，而不是 kubernetes-ingress 这个由 Nginx 公司维护的。

ingress-nginx 官网地址：https://kubernetes.github.io/ingress-nginx/

- **安装**

先应用配置：

```shell
wuxianmimi kubernetes-complex % kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx unchanged
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
wuxianmimi kubernetes-complex %
```

在视频中，接下来会使用 **minikube addons enable ingress** 命令，但是咪咪本地运行此命令会报错，网上找了很多解决办法试了试都无法解决问题。

而且经过测试，这条命令与上一条 kubectl apply 命令只要运行了其中一条，那剩下那条命令运行就会报错。

这里写继续往下走流程。

- **配置**

在 k8s 目录创建 ingress-service.yaml：

```yaml
# 从 Kubernetes 1.19 起改名了
# Kubernetes 1.22 开始只支持 networking.k8s.io/v1
# apiVersion: extensions/v1beta1
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: nginx
    # /api/ 的请求转发后重写，删除 /api 部分
    # 新的 apiVersion 需要使用这种写法来匹配 path 的正则
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - http:
        paths:
          - path: /(.*)
            # pathType 是必填字段
            pathType: Prefix
            # 由于 apiVersion 不同，格式变了
            backend:
              # serviceName: client-cluster-ip-service
              # servicePort: 3000
              service:
                name: client-cluster-ip-service
                port:
                  number: 3000
          - path: /api/(.*)
            pathType: Prefix
            backend:
              # serviceName: server-cluster-ip-service
              # servicePort: 5000
              service:
                name: server-cluster-ip-service
                port:
                  number: 5000
```

咪咪本地的 Kubernetes 版本是 v1.25.3，视频中使用的 apiVersion 已无法使用，所以配置格式有点不同。

在终端 **kubectl apply** 应用配置，随后通过 **minikube ip** 获取虚拟机 IP 地址，在浏览器访问该地址来访问我们的服务。

Mac 用户还是在终端输入 **minikube service** 命令来获取访问 URL：

```shell
wuxianmimi kubernetes-complex % minikube service ingress-nginx-controller --url -n ingress-nginx
http://127.0.0.1:62664
http://127.0.0.1:62665
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

访问 http://127.0.0.1:62664：

![img]()

启动成功，且可正常使用

尝试提交了几个值，可以正常运行。

- **minikube 控制台**

在命令行输入 minikube dashboard 可以打开控制台：

```shell
wuxianmimi ~ % minikube dashboard
🔌  正在开启 dashboard ...
    ▪ Using image docker.io/kubernetesui/dashboard:v2.7.0
    ▪ Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
💡  Some dashboard features require the metrics-server addon. To enable all features please run:

	minikube addons enable metrics-server


🤔  正在验证 dashboard 运行情况 ...
🚀  Launching proxy ...
🤔  正在验证 proxy 运行状况 ...
🎉  Opening http://127.0.0.1:59433/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

会自动打开页面：

![img](https://assets.ng-tech.icu/item/6e13e8686cc03f293c1cc2aebfba4d7ea9add88f.png@942w_561h_progressive.webp)minikube 控制台

# 第十六章：Kubernetes 生产环境部署（Google Cloud）

使用 Google Cloud 部署。

略。

# 第十七章：网站域名与证书配置

略。

# 第十八章：介绍 Skaffold

Skaffold 是一个命令行工具，可以处理构建、推送、部署等工作流。

官网链接：https://skaffold.dev/

视频中演示了在本地开发时提供监听，当代码改动时，自动更新 Kubernetes 中的相关对象。

# 第十九章：结语

略。
