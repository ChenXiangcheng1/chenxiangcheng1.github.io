---
title: Docker
date: 2023-10-24
update: 2023-10-24

tags:
categories:
  - container

keywords: Docker
description: 主要是基础的Docker命令、dockerfile
top_img:
cover: https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_01_15_30.png
---



# Docker

Docker Desktop = Docker Engine + Kubernetes + Compose V2

[官方文档](https://docs.docker.com/)	

Registry：[docker hub](https://hub.docker.com/search?q=)	|	[quay](https://quay.io/)



TODO：先学完 [dockertips](https://dockertips.readthedocs.io/en/latest/docker-image/get-image.html)+ali网盘网课，再看官方文档



## 概念

<table>
    <tr>
        <td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_04_23_25.png" alt="1" style="zoom: 25%;" /></center></td>
        <td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_04_23_28.png" alt="c2" style="zoom: 25%;" /></center></td>
    </tr>
    <tr>
    	<td><center>容器技术出现之前</center></td>
        <td><center>容器技术出现之后</center></td>
    </tr>
    <tr>
        <td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_01_15_30.png" alt="image-20230901153028373" style="zoom:80%;" /></center></td>
        <td><center><img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_04_23_27.png" alt="docker-image" style="zoom:50%;" /></center></td>
    </tr>
    <tr>
    	<td><center>Docker架构</center></td>
        <td><center>镜像/容器命令</center></td>
    </tr>
</table>


镜像是一个分层 read-only 文件，包含源码、依赖、系统库等运行 application 所需要的文件
容器是一个运行中的镜像，将应用程序与环境隔离

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting1/img/2023_09_04_23_26.png" alt="image-20230901152940157" style="zoom: 40%;" />



## 错误

```bash
$ docker version
error during connect: this error may indicate that the docker daemon is not running: Get "http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.24/version": open //./pipe/docker_engine: The system cannot find the file specified.

$ wsl -d arch
参考的对象类型不支持尝试的操作。
Error code: Wsl/Service/0x8007273d
```

连接期间的错误：此错误可能表明Docker守护程序未运行
解决：启动 Docker Desktop



问题：启动Docker Desktop报错Unexpected WSL error
解决：netsh winsock reset 后重启计算机，完成重置Winsock目录
原因：Docker Desktop 没正常关闭导致



## 1命令

docker+对象类型+具体操作command+参数+对象



### 其他

| docker 管理的对象                                            | 释义                               | 具体操作                                                     | 释义                                                         | 参数                                                         |
| ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| docker exec <CONTAINER><InnerCOMMAND><br />docker exec -it xxx bash | 在一个正在运行的容器中执行一个命令 | docker-entrypoint.sh 容器启动入口，在/bin目录中<br />sh、bash 命令解释器，在/bin目录中<br />exit退出 |                                                              | -i 交互的<br />-t 分配一个伪tty(终端)                        |
| docker version                                               | 显示docker版本信息                 |                                                              |                                                              |                                                              |
| docker info                                                  | 显示整个系统信息                   |                                                              |                                                              |                                                              |
| **docker system**                                            | 管理 Docker                        |                                                              |                                                              |                                                              |
|                                                              |                                    | prune                                                        | 删除未使用数据<br />删除所有未使用的**停止的容器**、**网络**、**dangling镜像**、**构建缓存**、卷(默认未选)<br />`docker image ls -f dangling=true` | -a 删除所有未使用的**停止的容器**、**网络**、**dangling镜像、未引用镜像**、**构建缓存**、卷(默认未选)<br />-f 不提示任何信息 |
| docker login                                                 | 登录到 registry                    |                                                              |                                                              |                                                              |
|                                                              |                                    |                                                              |                                                              |                                                              |



### image 镜像

<IMAGE>：name:tag(默认为latest)

| docker image <COMMAND>                          | 释义                                                         | 参数                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| pull                                            | 从 registry (公有public/私有private)下载镜像                 |                                                              |
| push <NAME>[:TAG]                               | 上传镜像到 Registry                                          |                                                              |
| save                                            | 导出镜像                                                     | -o 写入到一个文件                                            |
| load                                            | 导入镜像                                                     | -i 从tar归档中读取                                           |
| build <PATH\|URL> [build context]               | 从 Dockerfile 构建镜像<br /><PATH> 为Dockerfile目录<br />build context(默认为dockerfile所在目录)下的所有文件会被包含在镜像的根目录下，配合.dockerignore使用 | -t 指定\<name:tag><br />-f 指定路径PATH/Dockerfile           |
| ls                                              | 列出所有镜像                                                 |                                                              |
| inspect <IMAGE>                                 | 显示镜像详细信息，主要看Architecture                         |                                                              |
| rm                                              | 移除镜像                                                     |                                                              |
| tag \<SOURCE_IMAGE>[:TAG] \<TARGET_IMAGE>[:TAG] | 根据原镜像创建新镜像                                         |                                                              |
| history <IMAGE>                                 | 显示一个镜像的历史(层次结构)                                 |                                                              |
| prune                                           | 删除所有未使用的dangling镜像<br />`docker image ls -f dangling=true` | -a 删除所有未使用的dangling镜像、未引用镜像<br />-f 不提示任何信息 |



镜像的获取：

* `docker image pull` 从 registry

* `docker image build` 从 Dockerfile

* `docker image load` 从文件
* `docker container commit` 从容器的更改



#### 架构支持

**AMD64**(由x86-64改名)

ARM(苹果M系列芯片就是ARM架构)



#### scratch

https://hub.docker.com/_/scratch

scratch 镜像是一个空镜像，用于作为构建容器的起点(但更常用的是其他 Linux 镜像作为基础镜像)
无法 docker image pull 只可以在 Dockerfile 中引用它

```dockerfile
FROM scratch  # 无操作，不会在镜像中创建额外的层
ADD hello /
CMD ["/hello"]
```



#### busybox

https://hub.docker.com/_/busybox

开源的小型的UNIX工具集合，为小型系统或嵌入式系统提供环境

**Usage：**

```bash
docker run -it --rm busybox
```

```dockerfile
FROM busybox
COPY ./my-static-binary /my-static-binary
CMD ["/my-static-binary"]
```



#### alpine

https://hub.docker.com/_/alpine

轻量级的 Alpine Linux，musl lib(一个C库) + busybox，大小5M，用于生产

**Usage：**

```dockerfile
FROM alpine:3.14
RUN apk add --no-cache mysql-client
ENTRYPOINT ["mysql"]
```



#### mysql



#### clickhouse-server

```bash
docker run -d -p 8123:8123 -p 9000:9000 --name clickhouse-server --ulimit nofile=262144:262144 -v /mnt/e/Develop/docker_mount/clickhouse/config/config.xml:/etc/clickhouse-server/config.xml clickhouse/clickhouse-server:latest
```



#### dolphindb

```bash
docker run -d -p 8848:8848 --name dolphindb dolphindb/dolphindb:v3.00.0
```



#### cloudflared

```bash
docker container run --name cloudflared --restart unless-stopped cloudflare/cloudflared:latest tunnel --protocol http2 --no-autoupdate run --token eyJhIjoiMDI3N2U5YTQ2MjNiNDdlYjg1MzQ5M2I0MTNlOWFjYjQiLCJ0IjoiMjQxYjBjOGMtODliMi00YmE5LWI5ZTMtNTFmNTllOTI5MTNjIiwicyI6IllUTXpOekJtWTJFdFltUTBOeTAwTnpCbExUbGtOREV0T0dSbFpESTBNbVE0TmpZMiJ9
```



### container 容器

容器状态：Created、Up XX minutes、Exited(1) XX seconds age        // restarting

| docker container [arg] <COMMAND>           | 释义                                                         | 参数                                                         |
| ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ls                                         | 列出所有正在运行的容器                                       | -a 显示所有容器<br />-q 只显示容器IDs                        |
| run <image>  [command]                     | 从镜像创建并运行一个新容器，若镜像不存在则去 docker hub 拉取 | -d 分离(detach)模式在后台运行<br />-p 指定端口转发，将主机端口映射到容器端口，例如-p 8080:80，效果0.0.0.0:8080->80/tcp<br />-i 交互的<br />-t 分配一个伪tty(终端)<br />--name 分配一个名字给容器<br />--rm 当容器退出自动删除<br />-v <list> 挂载一个卷，例如 `-v volumename:/path`<br />-e 设置环境参数<br />--network 指定网络<br />--restart=always(on-failure:5、unless-stopped、默认no) 设置启动docker守护进程时的重启策略 |
| update <container>                         | 更新CPU设置、内存设置、restart策略                           | --restart=always(on-failure:5、unless-stopped、默认no)       |
| stop <container>                           | 暂停正在运行的容器                                           |                                                              |
| restart <container>                        | 重启已退出运行的容器                                         |                                                              |
| rm <container>                             | 移除容器                                                     |                                                              |
| logs [OPTIONS] CONTAINER                   | 获得一个容器的日志                                           | -f 跟踪日志输出                                              |
| top <container>                            | 显示正在运行的容器的进程                                     |                                                              |
| commit <container> [repository[:tag]]      | 从容器的更改创建镜像                                         |                                                              |
| inspect [OPTIONS] CONTAINER [CONTAINER...] | 显示容器的详细信息                                           | --format 使用自定义模板格式化输出<br />https://docs.docker.com/go/formatting/ |
| cp src_path dest_path                      |                                                              |                                                              |

docker container run示例(WSL Arch内)：`docker container run --name mariadb -p 3307:3306 -e MYSQL_ROOT_PASSWORD=Mariadb1610 -v /mnt/e/Develop/docker_mount/data/mariadb/data:/var/lib/mysql -d --restart unless-stopped mariadb:latest`



#### 批量操作

```bash
docker container rm $(docker container ls -aq)  # 批量操作
```



#### run 命令执行过程

`docker container run -d --p 80:80 --name webhost nginx`

1. 在本地**查找**是否有 nginx 这个 image **镜像**，若没有本地该镜像则去远程的 image registry 查找该最新版本镜像
   默认的 registry 是 Docker Hub，下载最新版本的 nginx 镜像(nginx:latest 默认)
2. 基于 nginx 镜像来**创建**一个新的**容器**，并且准备运行
3. docker engine **分配**给这个容器一个**虚拟 IP 地址**
4. 在宿主机上打开 80 端口并把容器的 80 **端口转发**到宿主机上
5. 启动容器，**运行指定的命令**(默认执行`docker-entrypoint.sh`shell脚本，去启动 nginx)



### volume 持久化数据的存储

[为什么需要数据卷，不直接挂载？](https://blog.csdn.net/qq_37960603/article/details/110135846)

[挂载笔记](https://www.jianshu.com/p/f97e14532c83)

* Data Volume, 存储位置**由Docker管理** (Linux 数据存储在/var/lib/docker/volumes/)

* Bind Mount，由**用户指定**数据存储的具体位置

创建的文件被保存在一个可写的容器层中

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1021_215416.png" alt="image-20231021215415956" style="zoom:60%;" />

| docker volume <COMMAND>   | 释义                     | 参数                                                         |
| ------------------------- | ------------------------ | ------------------------------------------------------------ |
| create [OPTIONS] [VOLUME] | 创建一个volume           | -d 指定volume驱动的名字(默认为local)<br />-o 设置驱动的参数选项(默认map[]) |
| ls                        |                          |                                                              |
| inspect                   | 显示一个volume的详细信息 |                                                              |
|                           |                          |                                                              |



### network

容器Ping宿主机：host.docker.internal

Drive: bridge、host、null



**底层实现的几个问题**

容器为什么能获取到IP地址？为什么容器之间的IP是互通的？
不同的Docker容器通过Linux的Network namespace进行了隔离，也就是不同的容器有各自的IP地址，路由表等，互不影响。

为什么容器能ping通外网？
iptables进行NAT，通过宿主机来访问外部网络

为什么外网能ping通容器内部？
需要端口转发，将容器内部的端口映射到宿主机的端口上

为什么宿主机可以ping通容器的IP？
宿主机可以访问到那个虚拟网络(虚拟网桥docker0)

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1023_233156.png" alt="docker-volume" style="zoom: 40%;" />

<center>Bridge桥接网络</center>

docker0 是虚拟网桥，连接容器和宿主机的网络
Docker就是通过iptables实现了容器和宿主机的网络隔离、请求转发 [Linux#iptables防火墙](../Linux/Linux.md#iptables 防火墙)

| docker network <COMMAND>            | 释义                   | 参数                                                         |
| ----------------------------------- | ---------------------- | ------------------------------------------------------------ |
| ls                                  |                        |                                                              |
| inspect                             |                        |                                                              |
| create                              | 创建一个网络           | -d 指定drive(默认bridge)<br />--gateway 指定主子网的网关(例如 172.18.0.1)<br />--subnet CIDR格式的子网(例如 172.18.0.0/16)表示一个网段 |
| rm                                  |                        |                                                              |
| connect [OPTIONS] NETWORK CONTAINER | 连接一个容器到一个网络 |                                                              |
| disconnect                          | 断连容器从一个网路     |                                                              |



#### 端口转发

<img src="https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_1024_155928.png" alt="image-20231024155928215" style="zoom:65%;" />

host 网络比 bridge 网络性能更好

host：容器与宿主机共享同一个网络命名空间



## 2Dockerfile

Dockerfile 是一个包含命令的 text 文档，用于自动化地构建镜像
默认文件名`Dockerfile`，使用其他文件名需要指定路径

[Dockerfile reference(重要)](https://docs.docker.com/engine/reference/builder/#dockerfile-reference)

[github 官方镜像](https://github.com/docker-library/official-images)

| 命令        | 释义                                                         | 建议                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| FROM        | 选择基础镜像                                                 | 固定版本tag而不是每次都使用latest<br />尽量选择体积小的镜像<br />Alpine Linux 会导致 Python Docker 镜像更大、构建时间更长 |
| RUN         | 执行指令                                                     | 每一行 RUN 命令都会产生一层 image layer                      |
| COPY        | 拷贝到目录                                                   | COPY app.html /path/                                         |
| ADD         | 自动解压缩 + 拷贝到目录                                      |                                                              |
| WORKDIR     | 指定容器的工作目录                                           | 容器启动时的当前目录<br />会自动创建不存在的目录             |
| ARG         | 构建参数                                                     | 配合 `docker image build xxxxxx --build-arg VERSION=2.0.0`<br />配合 ${VERSION} |
| ENV         | 环境变量                                                     |                                                              |
| CMD         | 容器启动时默认执行的命令                                     | `docker container run <image> [command]` 会覆盖CMD的命令<br />Shell格式：CMD echo "hello $NAME"<br />Exec格式：CMD ["sh", "-c", "echo hello $NAME"] |
| ENTRYPOINT  | 容器启动时一定会执行的命令                                   | 配合 `docker container run -it xxx xxarg` xxarg会作为参数传递给ENTRYPOINT |
| EXPOSE      | 暴露端口                                                     | EXPOSE 80，用于添加inspect信息 "Config.ExposedPorts": {"80/tcp": {}} |
| USER        | 指定用户(默认root用户)                                       |                                                              |
| VOLUME      |                                                              | VOLUME ["/app"]                                              |
| HEALTHCHECK | [dockerfile健康检查](https://docs.docker.com/reference/dockerfile/#healthcheck) | `HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:5000/ || exit 1` |



示例：

```dockerfile
FROM gcc:9.4 AS builder
COPY hello.c /src/hello.c
WORKDIR /src
RUN gcc --static -o hello hello.c

FROM alpine:3.13.5
COPY --from=builder /src/hello /src/hello
RUN groupadd -r xxgroup && \
	useradd -r -g xxgroup xxuser && \
    chown -R xxgroup:xxuser /src # change own
USER xxuser
ENTRYPOINT [ "/src/hello" ]
CMD []
```



### .dockerignore

build context(默认为dockerfile所在目录)下的所有文件会被包含在镜像的根目录下，使用.dockerignore排除

```.dockerignore
.vscode/
env/
```



## 3容器编排工具

指容器的自动化创建、配置、部署和管理工具



### Compose V2

(开发环境用Compose就好，生产环境用swarm k8s)

docker compose用于配置你的docker应用程序的服务、网络、卷等等，用于取代docker container run命令
默认文件名`目录名/docker-compose.yml`，使用其他文件名需要指定路径。创建的容器、网络名有前缀为目录名

[github](https://github.com/docker/compose)	|	[docker compose reference(重要)](https://docs.docker.com/compose/compose-file/)	|	[示例](https://github.com/docker/awesome-compose?tab=readme-ov-file)	|	[voting-app示例](https://github.com/dockersamples/example-voting-app)

[composerize](https://www.composerize.com/)

```yml
version: "3.8"

services: # 容器
  servicename: # 服务名字，这个名字也是内部 bridge网络可以使用的 DNS name
  	container_name:
  	build:
  	  context: ./flask  # 到该目录找DOCKERFILE
  	  dockerfile: Dockerfile.dev
    image: redis:latest  # 镜像的名字，如果本地镜像不存在则通过dockerfile构建
    restart: always
    command:  # 可选，如果设置，则会覆盖默认镜像里的 CMD命令
      - sh -c "while true; do sleep 3600; done"  
      - redis-server --requirepass {REDIS_PASSWORD}
    environment: # 可选，相当于 docker run里的 --env
      - REDIS_HOST=redis-server
      - REDIS_PASS={REDIS_PASSWORD}
    depends_on:  # 可选，服务依赖配置，影响服务启动顺序
      - servicename2
      flask:
        condition: service_healthy  # 服务依赖条件配置
    volumes: # 可选，相当于docker run里的 -v  # 很方便
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro  # ro表示readonly
      - ./var/log/nginx:/var/log/nginx
    networks: # 可选，相当于 docker run里的 --network
      - demo-network
      - mynetwork2
      - frontend
    ports: # 可选，相当于 docker run里的 -p
      - 8080:5000
    healthcheck:  # 健康检查，示例https://gist.github.com/phuysmans/4f67a7fa1b0c6809a86f014694ac6c3a
      test: ["CMD", "curl", "-f", "http://localhost:5000"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 40s
  servicename2:
  	network:
  	  - backend

volumes: # 可选，相当于 docker volume create

networks: # 可选，相当于 docker network create
  demo-network:
    ipam:
      driver: default
      config:
        - subnet: "172.16.238.0/24"
```

`.env` 文件，用于传入变量

```.env
REDIS_PASSWORD=abc123
```



#### 命令

docker compose up -d

| docker-compose <COMMAND> | 释义                                             | 参数                                                         |
| ------------------------ | ------------------------------------------------ | ------------------------------------------------------------ |
| up                       | 创建并启动容器                                   | -d Detached模式<br />-f 指定compose文件<br />--build 对已启动的服务重新构建<br />--remove-orphans 移除compose文件中未定义的容器<br />--scale 服务名=numb 水平扩展(有DNS解析和负载均衡)<br />--env-file ./myenv |
| stop                     | 停止服务                                         |                                                              |
| rm                       | 移除停止的服务容器                               |                                                              |
| restart                  | 重启服务容器                                     |                                                              |
|                          |                                                  |                                                              |
| pull                     | 拉取服务镜像                                     |                                                              |
| build                    | 构建服务从dockerfile                             |                                                              |
|                          |                                                  |                                                              |
| config                   | 显示并解析完整compose文件，若有错误会warning警告 |                                                              |
| logs                     | 显示容器的输出日志                               | -f 跟踪                                                      |
| ps                       | 显示容器                                         |                                                              |

docker会将container名字写入DNS
docker-compose会将服务名写入DNS



```bash
# 更新镜像
docker-compose pull
docker-compose up -d

# 自动更新：https://github.com/containrrr/watchtower
```





### docker swarm

不如k8s



### Kubernetes

适合用于生产环境



# Podman

技术选型：

[CRI(Container Runtime Interface)官网]( https://opencontainers.org/)

|      | docker(推荐)             | Podman                                      | macOS OrbStack(推荐) | nerdctl          |
| ---- | ------------------------ | ------------------------------------------- | -------------------- | ---------------- |
| 优点 | **生态好**               | 兼容docker-compose(也有在线转换podman run)  |                      | 底层是containerd |
| 缺点 | docker desktop商业要收费 | 存在build可能与docker不一致的问题(巨大劣势) |                      |                  |





# 我的安装

