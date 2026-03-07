---
title: AI开发者的Docker实践：汉化（中文），更换镜像源，Dockerfile，部署Python项目
date: 2026-01-01 08:03:00
tags:
  - "Docker"
  - "容器化"
  - "Python部署"
  - "汉化"
  - "镜像源"
categories:
  - "开发工具"
---

﻿@[TOC](AI开发者的Docker实践：汉化（中文），更换镜像源，Dockerfile，部署Python项目)
# Dcoker官网
[https://www.docker.com/](https://www.docker.com/)
![在这里插入图片描述](/img/f97943cda3f94a269f3bcd73c97e9844.png)


Docker 是一种开源的 容器化平台，用于构建、部署和运行应用程序。它通过容器技术将应用程序及其依赖项打包到一个可移植的单元中，从而实现跨环境的一致性。
在传统的项目开发中，开发者经常遇到环境不一致的问题，比如代码在本地开发环境运行正常，但在测试或生产环境却出现各种错误，原因可能是操作系统版本、依赖库版本或配置差异。此外，传统部署方式需要手动安装和配置各种软件环境，过程繁琐且容易出错，不同服务器之间的环境也难以保持一致。

Docker 的核心目标就是解决这些问题，通过容器化技术将应用及其运行环境打包在一起，确保应用在不同环境中表现一致。Docker 的出现极大简化了开发、测试和部署的流程，成为现代 DevOps 和云计算中的重要工具。Docker 有几个显著特点：

 - 轻量性：由于容器共享宿主机的操作系统内核，它们比传统虚拟机更小且启动更快，解决了传统虚拟化技术资源占用高、启动慢的问题。
 - 可移植性：Docker 容器可以在任何支持 Docker 的平台上运行，无论是本地开发机、物理服务器还是云环境，彻底解决了「在我机器上能跑，线上却不行」的难题。
 - 隔离性：每个容器拥有独立的文件系统、网络和进程空间，确保应用之间互不干扰，避免了传统部署中多个应用共用环境导致的依赖冲突问题。
 - 标准化：Docker 提供统一的接口和工具链，使得构建、分发和运行容器变得简单高效，替代了传统部署中复杂的手动配置流程。
![在这里插入图片描述](/img/244ee9c72ec040bba8c0847f96c2f30a.png)

## 1、核心概念
### 镜像 (Image)

 - 镜像是容器的静态模板，包含运行应用程序所需的所有文件和配置。
 - 镜像通过 Dockerfile 定义，并可以分层存储，便于复用和版本管理。
### 容器 (Container)

 - 容器是轻量级、独立的运行环境，包含应用程序及其所有依赖（代码、运行时、库、环境变量等）。
 - 容器基于镜像创建，每个容器相互隔离（进程、文件系统、网络等）。
### 仓库 (Repository)

 - 仓库用于存储和分发镜像，如 [Docker Hub](http://registry.hub.docker.com)（官方公共仓库）、私有仓库（如 Harbor）。

### Dockerfile

 - 一个文本文件，通过指令（如 FROM, COPY, RUN）定义如何构建镜像。

### Docker Compose

 - 用于定义和运行多容器应用的工具，通过 docker-compose.yml 文件配置服务、网络和卷。
## 2、Docker 的核心组件
### Docker 引擎（Docker Engine）

核心后台服务，包含：

 - Docker Daemon：管理容器、镜像、网络和卷。
 - REST API：与 Docker Daemon 交互的接口。
 - CLI：命令行工具（如 docker 命令）。

### Docker 网络

支持多种网络模式：

 - bridge（默认）：容器通过虚拟网桥通信。
 - host：容器直接使用主机网络。
 - overlay：用于跨主机的容器通信（如 Docker Swarm）。

### Docker 存储卷（Volume）

持久化数据存储机制，独立于容器生命周期。

## 3、Docker 的工作流程
### 开发阶段

 - 编写 Dockerfile 定义应用环境。
 - 使用 docker build 构建镜像。
 - 推送镜像到仓库（如 docker push）。

### 部署阶段

 - 从仓库拉取镜像（如 docker pull）。
 - 使用 docker run 启动容器。
# Windows 上的安装和汉化（中文）

**运行**：Docker 启动之后会在 Windows 任务栏出现鲸鱼图标。
![在这里插入图片描述](/img/e193066a156a4e2990975a174c36f77d.png)
## 1.下载对应版本中文语言包
[https://github.com/asxez/DockerDesktop-CN](https://github.com/asxez/DockerDesktop-CN)

![在这里插入图片描述](/img/1ceb8c9b158d40b3bc63684d4b3a1016.png)

## 2.下载 app-Windows-x86.asar

![在这里插入图片描述](/img/4453eb8c20c748959eccf21908dc8fdd.png)
## 3.找到docker安装目录`\Docker\Docker\frontend\resources`，备份并替换app.asar文件。
![在这里插入图片描述](/img/f2663af44473470abf8a5e2b6559967f.png)
## 4.重启Docker
![在这里插入图片描述](/img/feaee6296faa46cd8c96b75d3ce40b4f.png)


## 5.启动终端后，通过命令可以检查安装后的 Docker 版本。

```bash
PS C:\Users\74223> docker --version
Docker version 28.0.4, build b8034c0
PS C:\Users\74223> docker-compose --version
Docker Compose version v2.34.0-desktop.1
```

## 6.测试（Docker引擎新增国内源）
Docker 下载镜像默认从国外的官网下载，在国内需要通过代理访问 / 更换国内镜像源。

```bash
PS C:\Users\74223> docker run hello-world
Unable to find image 'hello-world:latest' locally
docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

![在这里插入图片描述](/img/76d7a26f3fc245fba0ff2de7bc9d9702.png)
将
```
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false
}
```
修改为

```
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "registry-mirrors": [
	"https://docker.1ms.run",
	"https://swr.cn-north-4.myhuaweicloud.com",
	"https://registry.cn-hangzhou.aliyuncs.com",
	"https://dockerproxy.com",
	"https://hub-mirror.c.163.com"
	]
}
```
重启

```bash
PS C:\Users\74223> docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
e6590344b1a5: Pull complete
Digest: sha256:c41088499908a59aae84b0a49c70e86f4731e588a737f1637e73c8c09d995654Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
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

# DockerFile

 - **定义**：DockerFile是一个文本文件，包含一系列指令（Instructions），用于定义如何构建 Docker 镜像。
 - **作用**：通过逐行执行指令，自动化构建镜像的层（Layer），最终生成一个可运行的容器环境。

## Dockerfile核心指令详解
![在这里插入图片描述](/img/03fb404367f64fc79e6886b04c3fc102.png)

### 1. FROM

 - 用途：指定基础镜像（必须为第一条指令）。

```bash
FROM ubuntu:22.04      # 使用官方 Ubuntu 镜像
FROM python:3.9-slim   # 使用轻量级 Python 镜像
```

### 2. RUN

 - 用途：在镜像构建过程中执行命令（如安装软件包）。

```bash
RUN apt-get update && apt-get install -y python3
```

### 3. COPY vs ADD

 - COPY：将文件从本地复制到容器中，功能类似于 ADD，但不支持远程 URL 和自动解压。

```bash
COPY requirements.txt /app/
```

 - ADD：将文件从本地复制到容器中，支持远程 URL 和自动解压。

```bash
ADD index.html /var/www/html/
```

### 4. WORKDIR

 - 用途：设置后续指令的工作目录（类似 cd）。

```bash
WORKDIR /app   # 后续指令默认在 /app 目录下执行
```

### 5. ENV

 - 用途：设置环境变量（可被后续指令和容器运行时使用）。

```bash
ENV NODE_ENV=production
ENV APP_PORT=8080
```

### 6. EXPOSE

 - 用途：声明容器运行时监听的端口（需通过 -p 映射到主机）。

```bash
EXPOSE 80   # 容器监听 80 端口
```

### 7. CMD vs ENTRYPOINT

 - CMD：定义容器启动时默认执行的命令。
 - 格式：CMD ["executable","param1","param2"] 或 CMD command param1 param2

```bash
CMD ["python3", "app.py"]
```

 - ENTRYPOINT：定义容器启动时运行的命令，与 CMD 不同的是，ENTRYPOINT 不会被覆盖。
 - 格式：ENTRYPOINT ["executable", "param1", "param2"]

```bash
ENTRYPOINT ["python3", "server.py"]
```

### 8. USER

 - 用途：指定运行容器时的用户（避免使用 root，提升安全性）。

```bash
RUN useradd -m appuser
USER appuser      # 后续指令以 appuser 身份运行
```

### 9. VOLUME
- 用途：声明数据卷挂载点（用于持久化数据）。  

```bash
VOLUME /data    # 容器运行时挂载到 /data
```

## Dockerfile示例

```bash
# 阶段1：构建依赖
FROM python:3.9 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# 阶段2：运行环境
FROM python:3.9-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .

# 确保 PATH 包含用户安装的包
ENV PATH=/root/.local/bin:$PATH

# 启动应用
CMD ["python", "app.py"]
```


```bash
# Base Images 
## 从天池基础镜像构建(from的base img 根据自己的需要更换，建议使用天池open list镜像链接：https://tianchi.aliyun.com/forum/postDetail?postId=67720) 
FROM registry.cn-shanghai.aliyuncs.com/tcc_public/python:3.10

##安装python依赖包 
RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r /app/requirements.txt

## 把当前文件夹里的文件构建到镜像的根目录下,并设置为默认工作目录 
ADD . /app

WORKDIR /app

## 镜像启动后统一执行 sh run.sh 
CMD ["sh", "/app/run.sh"]
```
**run.sh**

```bash
ls

cd /app; python ml_baseline.py predict
```
## 在Docker构建过程中，是否需要重新构建镜像取决于Dockerfile的指令顺序和文件变动情况。
### 1. Docker的层缓存机制
Docker镜像由多个层（Layer）组成，每个Dockerfile指令对应一个层。如果某一层及其之前的所有层未发生变化，Docker会直接使用缓存，避免重复执行耗时操作（如安装依赖）。若某一层变动，后续所有层缓存失效，需要重新构建。



# Docker 命令
![在这里插入图片描述](/img/999a6fb7140540269d056dce23836d88.png)

---
## 1. 镜像相关命令

| 命令 | 描述 |
|------|------|
| `docker images` | 列出本地所有的镜像。 |
| `docker pull <镜像名>:<标签>` | 从 Docker Hub 拉取指定镜像，例如：`docker pull ubuntu:latest`。 |
| `docker build -t <镜像名>:<标签> .` | 使用 Dockerfile 构建镜像，`.` 表示当前目录为上下文路径。 |
| `docker rmi <镜像ID>` | 删除指定的镜像。 |
| `docker tag <源镜像ID> <目标镜像名>:<标签>` | 为镜像打标签，便于推送或管理。 |

---

## 2. 容器相关命令

| 命令 | 描述 |
|------|------|
| `docker ps` | 列出所有正在运行的容器。 |
| `docker ps -a` | 列出所有容器（包括停止的）。 |
| `docker run [选项] <镜像名>` | 创建并启动一个新的容器。<br>常用选项：<br>`-d` 后台运行容器。<br>`-p` 端口映射，例如 `-p 8080:80`。<br>`-v` 卷挂载，例如 `-v /host/path:/container/path`。<br>`--name` 指定容器名称。 |
| `docker start <容器ID/名称>` | 启动已停止的容器。 |
| `docker stop <容器ID/名称>` | 停止正在运行的容器。 |
| `docker restart <容器ID/名称>` | 重启容器。 |
| `docker rm <容器ID>` | 删除指定的容器。 |
| `docker logs <容器ID/名称>` | 查看容器的日志输出。 |
| `docker exec -it <容器ID/名称> /bin/bash` | 进入正在运行的容器的交互式终端。 |

---

## 3. 数据卷相关命令

| 命令 | 描述 |
|------|------|
| `docker volume create <卷名>` | 创建一个新的数据卷。 |
| `docker volume ls` | 列出所有数据卷。 |
| `docker volume inspect <卷名>` | 查看数据卷的详细信息。 |
| `docker volume rm <卷名>` | 删除指定的数据卷。 |

---

## 4. 网络相关命令

| 命令 | 描述 |
|------|------|
| `docker network ls` | 列出所有网络。 |
| `docker network inspect <网络ID/名称>` | 查看网络的详细信息。 |
| `docker network create <网络名>` | 创建一个新的网络。 |
| `docker network connect <网络名> <容器名>` | 将容器连接到指定网络。 |
| `docker network disconnect <网络名> <容器名>` | 将容器从指定网络断开。 |

---

## 5. 其他常用命令

| 命令 | 描述 |
|------|------|
| `docker stats` | 实时查看容器的资源使用情况（CPU、内存等）。 |
| `docker system prune` | 清理未使用的容器、网络、镜像和卷。 |
| `docker save -o <文件名> <镜像名>` | 将镜像保存为 tar 文件。 |
| `docker load -i <文件名>` | 从 tar 文件加载镜像。 |
| `docker commit <容器ID> <新镜像名>` | 根据容器创建一个新的镜像。 |

---

## docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
常用参数说明：

 - -d: 后台运行容器并返回容器 ID。
 - -it: 交互式运行容器，分配一个伪终端。
 - --name: 给容器指定一个名称。
 - -p: 端口映射，格式为 host_port:container_port。
 - -v: 挂载卷，格式为 host_dir:container_dir。
 - --rm: 容器停止后自动删除容器。
 - --env 或 -e: 设置环境变量。
 - --network: 指定容器的网络模式。
 - --restart: 容器的重启策略（如 no、on-failure、always、unless-stopped）。
 - -u: 指定用户。

**交互式运行并分配终端**
```bash
docker run -it ubuntu /bin/bash
```
# Docker Compose
**Docker Compose** 是一个用于定义和运行多容器 Docker 应用程序的工具。通过一个 YAML 文件（`docker-compose.yml`）配置所有服务、网络和卷，实现一键启动/停止整个应用栈。

---

## 核心概念

1. **服务 (Service)**  
   每个容器对应一个服务（如 Web 服务器、数据库）。在配置文件中定义镜像、端口、环境变量等。
2. **项目 (Project)**  
   默认以当前目录命名，所有服务组成一个独立环境（可通过 `-p` 指定项目名）。
3. **网络 (Network)**  
   服务间通过自定义网络自动通信（默认创建 `projectname_default` 网络）。
4. **卷 (Volume)**  
   持久化数据（如数据库文件），避免容器重启丢失。

---

## 基础命令

| 命令                                  | 作用                                 |
| ------------------------------------- | ------------------------------------ |
| `docker compose up`                   | 启动所有服务（`-d` 后台运行）        |
| `docker compose down`                 | 停止并删除所有容器、网络             |
| `docker compose ps`                   | 查看运行中的服务                     |
| `docker compose logs [服务名]`        | 查看日志（`-f` 实时追踪）            |
| `docker compose build`                | 重新构建服务的镜像                   |
| `docker compose exec [服务名] [命令]` | 进入运行中的容器（如 `exec app sh`） |

---

### 
# 部署Python项目
## 1. 准备 Python 项目

```python
myapp/
├── app/
│   ├── __init__.py
│   └── main.py
├── requirements.txt
└── Dockerfile
```
![在这里插入图片描述](/img/ade1f59474214e6bacaa73425320a017.png)
**application.py**

```python
import sys

# yes, just adding two numbers
def add_two_numbers(a=0, b=0):
    result = a + b
    print(f"a is {a}")
    print(f"b is {b}")
    print(f"solution is {result}")


if __name__ == "__main__":
    if len(sys.argv) > 2:
        add_two_numbers(int(sys.argv[1]), int(sys.argv[2]))
    else:
        add_two_numbers()

```

## 2. 创建 Dockerfile（文件不带后缀）
![在这里插入图片描述](/img/21ced1ca13354373a6dbee48fb121575.png)
**requirements.txt**
![在这里插入图片描述](/img/8646672c8a0c45139f9ae02ec0d6e143.png)

## 3. 构建 Docker 镜像

```python
# -t 指定镜像名称和标签，. 表示Dockerfile所在的当前目录。
docker build -t my-python-app:1.0 .
```

```python
Windows PowerShell
版权所有（C） Microsoft Corporation。保留所有权利。

安装最新的 PowerShell，了解新功能和改进！https://aka.ms/PSWindows

PS C:\Users\15734> cd "D:\PyCharm Community Edition 2022.2.1\pythonProject\docker-tutorial"     
PS D:\PyCharm Community Edition 2022.2.1\pythonProject\docker-tutorial> docker build -t my-python-app:1.0 .
[+] Building 8.6s (8/8) FINISHED                                           docker:desktop-linux 
 => [internal] load build definition from Dockerfile                                       0.1s
 => => transferring dockerfile: 376B                                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim                         1.7s
 => [internal] load .dockerignore                                                          0.0s
 => => transferring context: 2B                                                            0.0s 
 => [internal] load build context                                                          0.2s 
 => => transferring context: 21.14kB                                                       0.1s 
 => [1/4] FROM docker.io/library/python:3.9-slim@sha256:9aa5793609640ecea2f06451a0d6f3793  0.0s 
 => CACHED [2/4] WORKDIR /app                                                              0.0s
 => [3/4] COPY . .                                                                         0.2s 
 => ERROR [4/4] RUN pip install --no-cache-dir -r requirements.txt                         6.2s
------
 > [4/4] RUN pip install --no-cache-dir -r requirements.txt:
3.132 Processing /C:/ci/aiohttp_1646806572557/work
3.135 ERROR: Could not install packages due to an OSError: [Errno 2] No such file or directory: '/C:/ci/aiohttp_1646806572557/work'
3.135
3.750
3.750 [notice] A new release of pip is available: 23.0.1 -> 25.0.1
3.750 [notice] To update, run: pip install --upgrade pip
------
Dockerfile:12
--------------------
  10 |     
  11 |     # 安装项目依赖
  12 | >>> RUN pip install --no-cache-dir -r requirements.txt
  13 |
  14 |     ENTRYPOINT ["python", "-u", "./application.py" ]
not complete successfully: exit code: 1
-------------------------------------------------------------------------------------------------XXView build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/vlp8jhik1yukve561d6ahwzg5
PS D:\PyCharm Community Edition 2022.2.1\pythonProject\docker-tutorial> docker build -t my-python-app:1.0 .rm Community Edition 2022.2.1\pythonProject\docker-tutorial>
[+] Building 12.6s (8/8) FINISHED                                          docker:desktop-linux  => [internal] load build definition from Dockerfile                                       0.1s 
 => => transferring dockerfile: 376B                                                       0.0s 
 => [internal] load metadata for docker.io/library/python:3.9-slim                         1.3s 
 => [internal] load .dockerignore                                                          0.1s
 => => transferring context: 2B                                                            0.0s
 => [internal] load build context                                                          0.2s
 => => transferring context: 3.46kB                                                        0.1s 
 => [1/4] FROM docker.io/library/python:3.9-slim@sha256:9aa5793609640ecea2f06451a0d6f3793  0.0s 
 => CACHED [2/4] WORKDIR /app                                                              0.0s
 => [3/4] COPY . .                                                                         0.3s
 => ERROR [4/4] RUN pip install --no-cache-dir -r requirements.txt                        10.0s
------
 > [4/4] RUN pip install --no-cache-dir -r requirements.txt:
5.994 Collecting alembic==1.13.1
6.348   Downloading alembic-1.13.1-py3-none-any.whl (233 kB)
6.623      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 233.4/233.4 kB 904.0 kB/s eta 0:00:00
6.820 ERROR: Could not find a version that satisfies the requirement anaconda-navigator==2.1.4 (from versions: none)
6.820 ERROR: No matching distribution found for anaconda-navigator==2.1.4
7.450
7.450 [notice] A new release of pip is available: 23.0.1 -> 25.0.1
7.450 [notice] To update, run: pip install --upgrade pip
------
Dockerfile:12
--------------------
  10 |     
  11 |     # 安装项目依赖
  12 | >>> RUN pip install --no-cache-dir -r requirements.txt
  13 |
  14 |     ENTRYPOINT ["python", "-u", "./application.py" ]
--------------------
ERROR: failed to solve: process "/bin/sh -c pip install --no-cache-dir -r requirements.txt" did not complete successfully: exit code: 1

View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/73hqsam0a7fmp9ep2zmd4pt6r
PS D:\PyCharm Community Edition 2022.2.1\pythonProject\docker-tutorial> docker build -t my-python-app:1.0 .
[+] Building 8.9s (8/8) FINISHED                                           docker:desktop-linux
 => [internal] load build definition from Dockerfile                                       0.0s
 => => transferring dockerfile: 376B                                                       0.0s 
 => [internal] load metadata for docker.io/library/python:3.9-slim                         0.7s 
 => [internal] load .dockerignore                                                          0.0s
 => => transferring context: 2B                                                            0.0s 
 => [1/4] FROM docker.io/library/python:3.9-slim@sha256:9aa5793609640ecea2f06451a0d6f3793  0.0s 
 => [internal] load build context                                                          0.0s 
 => => transferring context: 512B                                                          0.0s 
 => CACHED [2/4] WORKDIR /app                                                              0.0s
 => [3/4] COPY . .                                                                         0.2s 
 => ERROR [4/4] RUN pip install --no-cache-dir -r requirements.txt                         7.6s
------
 > [4/4] RUN pip install --no-cache-dir -r requirements.txt:
3.397 Collecting alembic==1.13.1
3.974   Downloading alembic-1.13.1-py3-none-any.whl (233 kB)
4.380      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 233.4/233.4 kB 586.9 kB/s eta 0:00:00
4.528 ERROR: Could not find a version that satisfies the requirement anaconda-navigator==2.1.4 (from versions: none)
4.528 ERROR: No matching distribution found for anaconda-navigator==2.1.4
5.084
5.084 [notice] A new release of pip is available: 23.0.1 -> 25.0.1
5.084 [notice] To update, run: pip install --upgrade pip
------
Dockerfile:12
--------------------
  10 |     
  11 |     # 安装项目依赖
  12 | >>> RUN pip install --no-cache-dir -r requirements.txt
  13 |
  14 |     ENTRYPOINT ["python", "-u", "./application.py" ]
--------------------
ERROR: failed to solve: process "/bin/sh -c pip install --no-cache-dir -r requirements.txt" did not complete successfully: exit code: 1

View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/q8at9hhjtj61fm40aclyopwgv
PS D:\PyCharm Community Edition 2022.2.1\pythonProject\docker-tutorial> docker build -t my-python-app:1.0 .
[+] Building 310.8s (7/8)                                                  docker:desktop-linux
 => [internal] load build context                                                          0.0s
[+] Building 311.0s (7/8)                                                  docker:desktop-linux 
 => [internal] load build context                                                          0.0s 
[+] Building 311.1s (7/8)                                                  docker:desktop-linux 
 => [internal] load build context                                                          0.0s 
[+] Building 322.5s (7/8)                                                  docker:desktop-linux
 => [internal] load build context                                                          0.0s 
[+] Building 322.7s (7/8)                                                  docker:desktop-linux
 => [internal] load build context                                                          0.0s 
[+] Building 322.8s (7/8)                                                  docker:desktop-linux
 => [internal] load build context                                                          0.0s 
[+] Building 894.6s (9/9) FINISHED                                         docker:desktop-linux 
 => [internal] load build definition from Dockerfile                                       0.0s 
 => => transferring dockerfile: 376B                                                       0.0s 
 => [internal] load metadata for docker.io/library/python:3.9-slim                         0.6s 
 => [internal] load .dockerignore                                                          0.0s 
 => => transferring context: 2B                                                            0.0s 
 => [1/4] FROM docker.io/library/python:3.9-slim@sha256:9aa5793609640ecea2f06451a0d6f3793  0.0s 
 => [internal] load build context                                                          0.0s 
 => => transferring context: 130B                                                          0.0s 
 => CACHED [2/4] WORKDIR /app                                                              0.0s 
 => [3/4] COPY . .                                                                         0.3s 
 => [4/4] RUN pip install --no-cache-dir -r requirements.txt                             886.0s 
 => exporting to image                                                                     6.9s
 => => exporting layers                                                                    6.8s 
 => => writing image sha256:5c80cd625d69fbfc8e6ada4d822165bad06666bf417db842bdfdc1c0da225  0.0s 
 => => naming to docker.io/library/my-python-app:1.0                                       0.0s 

View build details: docker-desktop://dashboard/build/desktop-linux/desktop-linux/njmlcpgxaeiewkez9b2nurns6
PS D:\PyCharm Community Edition 2022.2.1\pythonProject\docker-tutorial> 
```

## 4. 运行容器
**查看镜像**

```python
 docker images
```

```python
PS D:\PyCharm Community Edition 2022.2.1\pythonProject\docker-tutorial>  docker images
REPOSITORY      TAG       IMAGE ID       CREATED              SIZE
my-python-app   1.0       5c80cd625d69   About a minute ago   872MB
my-python-app   latest    1e9870b0df05   18 hours ago         126MB
```

```python
# 基础运行
docker run -d -p 8000:8000 --name myapp my-python-app:1.0

# 带环境变量
docker run -d -p 8000:8000 \
  -e DEBUG_MODE=1 \
  --name myapp \
  my-python-app:1.0

# 挂载卷（开发时同步代码）
docker run -d -p 8000:8000 \
  -v $(pwd)/app:/app/app \
  --name myapp \
  my-python-app:1.0
```
-d 参数表示容器在后台运行，-p 参数用于将主机的8000端口映射到容器的8000端口。

```python
PS D:\PyCharm Community Edition 2022.2.1\pythonProject\docker-tutorial> docker run -d -p 8000:8000 --name myapp my-python-app:1.0
4af9469d1cc95789e6c81022779dc52cee604c6e5e325b7d1f7b977c71b7f8e8
```
# 人工智能竞赛-阿里云镜像实战

## 一、创建阿里云镜像仓库sais_synthetic
### 1️⃣ 登录阿里云账号
![在这里插入图片描述](/img/0ff095934c2a48b285315e27ce0370db.png)
![在这里插入图片描述](/img/ec5143a78bb747b190d4995f12c4c144.png)

### 2️⃣ 创建个人版实例
![在这里插入图片描述](/img/f9e183b98b8a42e89f42174f055096fd.png)
![在这里插入图片描述](/img/82256d9c80a74ef18cd6d6a5f68eb345.png)
![在这里插入图片描述](/img/5eae621c2f0e421499ebe04b54053148.png)
![在这里插入图片描述](/img/2eba32070a7644f3a90bc332afc6c6f1.png)
![在这里插入图片描述](/img/05e9cec2ed274e34afd1977e3662c552.png)
![在这里插入图片描述](/img/94bc5eac64934f288094fca0088b33c1.png)
#### 选择本地仓库
##### 1. 点击【创建镜像仓库】
![在这里插入图片描述](/img/43d5812156be472b8cc3ba82f9a54a98.png)
##### 2. 设置仓库信息
**这里建议选择私有更安全**
![在这里插入图片描述](/img/352cd696f33b4e1ba0e38b278a592d55.png)
##### 3. 选择【本地仓库】
![在这里插入图片描述](/img/ed16babbb6264ce8a4d879c0affd9146.png)
##### 4. 创建成功！
![在这里插入图片描述](/img/4bbb1b53316e459daeba73062e9d8e9f.png)
## 二、打包镜像并推送
### 1️⃣ 打开终端，跳转到下载好的文件目录下
**cd + 直接拖拽或复制文件路径，回车跳转**
![在这里插入图片描述](/img/410d65745bd145afb7ec5ec158eb72a5.png)
### 2️⃣ 登录ACR仓库

**注意输入密码时不会显示、直接输入然后回车即可
当看到“Login Succeeded”即为成功**

![在这里插入图片描述](/img/d2086981ed0d43a8af47c6f92650abf0.png)
### 3️⃣ 本地打包镜像（约5分钟）

**使用命令 `docker build -t wind_pv:v1 .`
千万不要忘记 “.”（代表当前目录）**

![在这里插入图片描述](/img/e7eacb0cd2e64547be5167d6724a8e92.png)
![在这里插入图片描述](/img/1ed0048aab7e414c9c8016b3a772101a.png)
### 4️⃣ 打tag（此步骤可省略）
**替换`[ImageId]`为 ` wind_pv:v1`
替换[镜像版本号]为 `v1`**
![在这里插入图片描述](/img/c9d9ac9274654070b34239ba76d79b28.png)
![在这里插入图片描述](/img/552b17e4e8c340f4b514c19083fa8dc2.png)
`docker images`命令可以查看当前环境下的docker镜像

![在这里插入图片描述](/img/605d10377222428fb2e2f6f85f3660e6.png)
### 5️⃣ 推送镜像
替换`[镜像版本号]`为 `v1`
![在这里插入图片描述](/img/24ef0552bae247818e22438965ed30a8.png)
![在这里插入图片描述](/img/ee6aa7334663420e827afdb69356527e.png)
## 三、提交赛事
### 1️⃣ 复制镜像仓库公网地址
#### 镜像版本正常
![在这里插入图片描述](/img/eefbad13828e4b0fbee655c25cb32bd5.png)
#### 复制公网地址
![在这里插入图片描述](/img/6b5f2a6064a743468d3ecc83b09b383b.png)
### 2️⃣ 提交赛事
**需要提交的地址 = 公网地址 + ":v1"**
#### 1. 粘贴公网地址，追加“:v1”
**如果是私有地址须填写用户名和密码**
![在这里插入图片描述](/img/b3806a7e1ac14b87b60822d3a138be97.png)
#### 2. 提交
![在这里插入图片描述](/img/7620a420c0b844d59c7388b7e2bed624.png)
## 四、总结
### CMD
```python
cd "D:\PyCharm 2025.1\PyProject\new_energy_predict"

docker login --username=*** crpi-******.cn-beijing.personal.cr.aliyuncs.com

docker build -t crpi-******.cn-beijing.personal.cr.aliyuncs.com/new_energy/windpv:v1 .

# docker tag windpv:v1 ******.cn-beijing.personal.cr.aliyuncs.com/new_energy/windpv:v1

docker push crpi-******.cn-beijing.personal.cr.aliyuncs.com/new_energy/windpv:v1
# 复制粘贴
crpi-******.cn-beijing.personal.cr.aliyuncs.com/new_energy/windpv:v1
# 可自己在本地测试运行
docker run crpi-******.cn-beijing.personal.cr.aliyuncs.com/new_energy/windpv:v1
```


![在这里插入图片描述](/img/5cb294a6e6424a84bb8fed37dd372ced.png)
### Dockfile

```xml
# Base Images 
## 从天池基础镜像构建(from的base img 根据自己的需要更换，建议使用天池open list镜像链接：https://tianchi.aliyun.com/forum/postDetail?postId=67720) 
FROM registry.cn-shanghai.aliyuncs.com/tcc_public/python:3.10

## 把当前文件夹里的文件构建到镜像的根目录下,并设置为默认工作目录
ADD . /app

WORKDIR /app

##安装python依赖包
RUN pip3 install numpy==1.26.4 pandas==2.2.2 scikit-learn==1.5.1 lightgbm==4.5.0 xgboost==2.1.1 catboost==1.2.6 netCDF4==1.7.2 --index-url=http://mirrors.aliyun.com/pypi/simple/ --trusted-host=mirrors.aliyun.com

## 镜像启动后统一执行 sh run.sh 
CMD ["sh", "/app/run.sh"]
```
### run.sh

```powershell
ls

cd /app; python WIND2PV.py qzk
```
# 常规技巧

 1. 检查基础镜像软件源和pip源是否替换为国内源，如果非国内源后续每次构建镜像会比较浪费时间。
 2. 必备软件包可直接安装于基础镜像内，以减少每次构建镜像时都要安装一遍的等待时间。
 3. 镜像面临调试问题时，可交互式进入容器后直接调试修改，直到成功后退出再在Dockerfile中修改。
 4. 养成使用Dockerfile的习惯，不要依赖于commit
 5. 每次镜像修改都给定新的版本号或标签，方便区分版本管理，有意义的版本最好使用有含义的字符作为版本号，如：frist_submit
 6. **深度学习常用镜像集合（包含国内源和海外源）[https://tianchi.aliyun.com/forum/postDetail?postId=67720](https://tianchi.aliyun.com/forum/postDetail?postId=67720)**

# 参考
1.[AI开发者的Docker实践](https://tianchi.aliyun.com/course/351?spm=a2c22.29205803.J_9902613000.6.7f5b36d97HVqKv)
2.[Docker 命令大全](https://www.runoob.com/docker/docker-command-manual.html?spm=a2c6h.13046898.publish-article.42.10f16ffa9SGo2L)
3.[项目依赖的python包requirements.txt文件的生成与安装](https://blog.csdn.net/qq_45832050/article/details/126789897?spm=1011.2415.3001.5331)
4.[零基础入门Docker-cuda练习场](https://tianchi.aliyun.com/competition/entrance/531863/customize253)
5.[魔搭社区](https://modelscope.cn/my/mynotebook)
6.[Codeup](https://codeup.aliyun.com/)
7.[Docker 万字教程：从入门到掌握](https://www.datawhale.cn/article/332)
8.[AI开发者的docker实践电子书](https://dockerpractice.readthedocs.io/zh/latest/dockerai/?spm=a2c22.12281976.0.0.1a221b05HYk58h)
9.[天池大赛docker提交演示](https://tianchi.aliyun.com/course/live/1610?spm=a2c22.12281976.0.0.1a221b05PQ9rdE)
10.[DataWhale春训营](https://www.datawhale.cn/learn/content/141/3630)
11.[https://github.com/coze-dev/coze-studio/blob/main/README.zh_CN.md](https://github.com/coze-dev/coze-studio/blob/main/README.zh_CN.md)
