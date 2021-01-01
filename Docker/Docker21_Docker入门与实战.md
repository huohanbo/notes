# 参考资料
> [Docker技术入门与实战（第3版）](https://weread.qq.com/web/reader/57f327107162732157facd6kc81322c012c81e728d9d180)

# Docker简介
如果说主机时代比拼的是单个服务器物理性能（如CPU主频和内存）的强弱，那么在云时代，最为看重的则是凭借虚拟化技术所构建的集群处理能力。

传统来看，虚拟化既可以通过硬件模拟来实现，也可以通过操作系统软件来实现。
而容器技术则更为优雅，它充分利用了操作系统本身已有的机制和特性，可以实现远超传统虚拟机的轻量级虚拟化。

## Docker是什么
Docker是基于Go语言实现的开源容器项目。

Docker的构想是要实现“Build, Ship and Run Any App, Anywhere”。
即通过对应用的封装（Packaging）、分发（Distribution）、部署（Deployment）、运行（Runtime）生命周期进行管理，达到应用组件级别的“一次封装，到处运行”。

Docker借鉴了Linux容器（Linux Containers, LXC）技术。

可以将Docker容器理解为一种轻量级的沙盒（sandbox）。
每个容器内运行着一个应用，不同的容器相互隔离，容器之间也可以通过网络互相通信。容器的创建和停止十分快速，几乎跟创建和终止原生应用一致；
另外，容器自身对系统资源的额外需求也十分有限，远远低于传统虚拟机。
很多时候，甚至直接把容器当作应用本身也没有任何问题。

## 为什么是Docker
+ Docker容器虚拟化的优势。通过容器来打包应用，解耦应用和运行平台。
+ Docker开发和运维中的优势。更快速的交付和部署，更高效的资源利用，更轻松的迁移和扩展，更简单的更新管理。
+ Docker与虚拟机比较的优势。Docker容器启动更快，Docker容器对系统资源需求更少，Docker通过类似Git设计理念的操作来方便用户获取、分发和更新应用镜像，Docker通过Dockerfile支持灵活的自动化创建和部署机制。

## Docker与虚拟化
虚拟化（virtualization）技术是一个通用的概念，在不同领域有不同的理解。
在计算领域，一般指的是计算虚拟化（computing virtualization），或通常说的服务器虚拟化。

虚拟化技术可分为基于硬件的虚拟化和基于软件的虚拟化。
基于软件的虚拟化从对象所在的层次，又可以分为应用虚拟化和平台虚拟化（通常说的虚拟机技术即属于这个范畴）。

Docker以及其他容器技术都属于操作系统虚拟化这个范畴，操作系统虚拟化最大的特点就是不需要额外的supervisor支持。

传统方式是在硬件层面实现虚拟化，需要有额外的虚拟机管理应用和虚拟机操作系统层。
Docker容器是在操作系统层面上实现虚拟化，直接复用本地主机的操作系统，因此更加轻量级。

# Docker核心概念
Docker大部分的操作都围绕着它的三大核心概念：镜像、容器和仓库。

## Docker镜像
Docker镜像Docker镜像类似于虚拟机镜像，可以将它理解为一个只读的模板。

镜像是创建Docker容器的基础。

## Docker容器
Docker容器类似于一个轻量级的沙箱，Docker利用容器来运行和隔离应用。

容器是从镜像创建的应用运行实例。它可以启动、开始、停止、删除，而这些容器都是彼此相互隔离、互不可见的。

镜像自身是只读的。容器从镜像启动的时候，会在镜像的最上层创建一个可写层。

## Docker仓库
Docker仓库类似于代码仓库，是Docker集中存放镜像文件的场所。

根据所存储的镜像公开分享与否，Docker仓库可以分为公开仓库（Public）和私有仓库（Private）两种形式。

有时候我们会将Docker仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。
实际上，仓库注册服务器是存放仓库的地方，其上往往存放着多个仓库。
每个仓库集中存放某一类镜像，往往包括多个镜像文件，通过不同的标签（tag）来进行区分。

# 安装Docker引擎
Docker引擎是使用Docker容器的核心组件，可以在主流的操作系统和云平台上使用，
包括Linux操作系统（如Ubuntu、Debian、CentOS、Redhat等）, macOS和Windows操作系统，以及IBM、亚马逊、微软等知名云平台。

可以访问Docker官网的[Get Docker 页面](https://www.docker.com/get-docker)，查看获取Docker的方式，以及Docker支持的平台类型。

Docker支持Docker引擎、Docker Hub、Docker Cloud等多种服务。
+ Docker引擎：包括支持在桌面系统或云平台安装Docker，以及为企业提供简单安全弹性的容器集群编排和管理；
+ DockerHub：官方提供的云托管服务，可以提供公有或私有的镜像仓库；
+ DockerCloud：官方提供的容器云服务，可以完成容器的部署与管理，可以完整地支持容器化项目，还有CI、CD功能。

Docker引擎目前分为两个版本：社区版本（Community Edition, CE）和企业版本（Enterprise Edition, EE）。

# 使用Docker镜像
Docker运行容器前需要本地存在对应的镜像，如果镜像不存在，Docker会尝试先从默认镜像仓库下载（默认使用Docker Hub公共注册服务器中的仓库），用户也可以通过配置，使用自定义的镜像仓库。

## 查看镜像
### 查看镜像列表
> docker images 或 docker image ls

```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
openoffice-demo     0.0.1               30355afeecf6        2 days ago          2.15GB
ubuntu              latest              1d622ef86b13        6 weeks ago         73.9MB
centos7.6-jre1.8    latest              31547ffd53a3        3 months ago        470MB
training/webapp     latest              6fae60ef3446        5 years ago         349MB
```
展示信息：
+ 来自于哪个仓库，比如ubuntu表示ubuntu系列的基础镜像；
+ 镜像的标签信息，比如18.04、latest表示不同的版本信息。标签只是标记，并不能标识镜像内容；
+ 镜像的ID（唯一标识镜像），如果两个镜像的ID相同，说明它们实际上指向了同一个镜像，只是具有不同标签名称而已；
+ 创建时间，说明镜像最后的更新时间；
+ 镜像大小，优秀的镜像往往体积都较小。

参数：
+ -a, --all=true|false：列出所有（包括临时文件）镜像文件，默认为否；
+ --digests=true|false：列出镜像的数字摘要值，默认为否；
+ -f, --filter=[]：过滤列出的镜像，如dangling=true只显示没有被使用的镜像；也可指定带有特定标注的镜像等；
+ --format="TEMPLATE"：控制输出格式，如．ID代表ID信息，.Repository代表仓库信息等；
+ --no-trunc=true|false：对输出结果中太长的部分是否进行截断，如镜像的ID信息，默认为是；
+ -q, --quiet=true|false：仅输出ID信息，默认为否。

### 查看镜像详细信息
可以查看制作者、适应架构、各层的数字摘要等。

> docker [image] inspect REPOSITORY

```
$ docker inspect ubuntu:latest
[
    {
        "Id": "sha256:1d622ef86b138c7e96d4f797bf5e4baca3249f030c575b9337638594f2b63f01",
        "RepoTags": [
            "ubuntu:latest"
        ],
        "RepoDigests": [
            "ubuntu@sha256:747d2dbbaaee995098c9792d99bd333c6783ce56150d1b11e333bbceed5c54d7"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2020-04-24T01:07:51.928109369Z",
        "Container": "8f0a86e65abdb09f6eeffc565fe6cf5ddf1213b5b14ce1ec92caa0347ee56901",
        "ContainerConfig": {
            "Hostname": "8f0a86e65abd",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/bash\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:8a6a781e255205d6bf9b529aea2aad5ad73780863dc4efaa62501dd806b5ed57",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
		
        "DockerVersion": "18.09.7",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:8a6a781e255205d6bf9b529aea2aad5ad73780863dc4efaa62501dd806b5ed57",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 73852122,
        "VirtualSize": 73852122,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/6d9fd270b42a9e8dce3737545299bed7e4507525581561f4c014d83e0620301f/diff:/var/lib/docker/overlay2/bf009dbce6f0e589761d5d133e662a35e40c40a3850743e13f7d09a09d725667/diff:/var/lib/docker/overlay2/71c6375601d3ef24cdaf6f934cca11c03fd1ef5985b6b21beca505930bdc51be/diff",
                "MergedDir": "/var/lib/docker/overlay2/62477e7312799f6ebab2007a5fefaa8ef6bc56ef48a7cd6b371acb1dbbef869a/merged",
                "UpperDir": "/var/lib/docker/overlay2/62477e7312799f6ebab2007a5fefaa8ef6bc56ef48a7cd6b371acb1dbbef869a/diff",
                "WorkDir": "/var/lib/docker/overlay2/62477e7312799f6ebab2007a5fefaa8ef6bc56ef48a7cd6b371acb1dbbef869a/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:7789f1a3d4e9258fbe5469a8d657deb6aba168d86967063e9b80ac3e1154333f",
                "sha256:9e53fd4895597d04f8871a68caea4c686011e1fbd0be32e57e89ada2ea5c24c4",
                "sha256:2a19bd70fcd4ce7fd73b37b1b2c710f8065817a9db821ff839fe0b4b4560e643",
                "sha256:8891751e0a1733c5c214d17ad2b0040deccbdea0acebb963679735964d516ac2"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

### 查看镜像历史
既然镜像文件由多个层组成，那么怎么知道各个层的内容具体是什么呢？这时候可以使用history子命令，该命令将列出各层的创建信息。

> docker [image] history REPOSITORY

```
$ docker history ubuntu:latest
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
1d622ef86b13        6 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>           6 weeks ago         /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B
<missing>           6 weeks ago         /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   811B
<missing>           6 weeks ago         /bin/sh -c [ -z "$(apt-get indextargets)" ]     1.01MB
<missing>           6 weeks ago         /bin/sh -c #(nop) ADD file:a58c8b447951f9e30…   72.8MB
```

## 创建镜像
创建镜像的方法主要有三种：基于已有镜像的容器创建、基于本地模板导入、基于Dockerfile创建。

### 基于已有容器创建
> docker [container] commit [OPTIONS] CONTAINER [REPOSITORY [:TAG]]

参数：
+ -a, --author=""：作者信息；
+ -c, --change=[]：提交的时候执行Dockerfile指令，包括CMD|ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|VOLUME|WORKDIR等；
+ -m, --message=""：提交消息；
+ -p, --pause=true：提交时暂停容器运行。

```
$ docker [container] commit -m "Added a new file" -a "Docker Newbee" a925cb40b3f0
```

### 基于本地模板导入
> docker [image] import [OPTIONS] file|URL|-[REPOSITORY [:TAG]]

```
$ cat ubuntu-18.04-x86_64-minimal.tar.gz | docker import - ubuntu:18.04
```

### 基于Dockerfile创建
基于Dockerfile创建是最常见的方式。Dockerfile是一个文本文件，利用给定的指令描述基于某个父镜像创建新镜像的过程。
> docker [image] build

Dockerfile：
```
FROM debian:stretch-slim

LABEL version="1.0" maintainer="docker user <docker_user@github>"

RUN apt-get update && \
	apt-get install -y python3 && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/＊
```

```
$ docker [image] build -t python:3 .
...
Successfully built 4b10f46eacc8
Successfully tagged python:3
$ docker images|grep python
python 3 4b10f46eacc8 About a minute ago    95.1MB
```

## 删除镜像
> docker rmi 或 docker image rm
> docker rmi IMAGE [IMAGE...]
参数：
+ -f, -force：强制删除镜像，即使有容器依赖它；
+ -no-prune：不要清理未带标签的父镜像。

## 清理镜像
使用Docker一段时间后，系统中可能会遗留一些临时的镜像文件，以及一些没有被使用的镜像，可以通过dockerimage prune命令来进行清理。
> docker [image] prune

参数：
+ -a, -all：删除所有无用镜像，不光是临时镜像；
+ -filter filter：只清理符合给定过滤器的镜像；
+ -f, -force：强制删除镜像，而不进行提示确认。

```
$ docker image prune -f
...
Total reclaimed space: 1.4 GB
```

## 添加镜像标签
为了方便在后续工作中使用特定镜像，还可以使用docker tag命令来为本地镜像任意添加新的标签。

> docker tag ubuntu:latest myubuntu:latest

```
$ docker tag ubuntu:latest myubuntu:latest
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
openoffice-demo     0.0.1               30355afeecf6        2 days ago          2.15GB
myubuntu            latest              1d622ef86b13        6 weeks ago         73.9MB
ubuntu              latest              1d622ef86b13        6 weeks ago         73.9MB
centos7.6-jre1.8    latest              31547ffd53a3        3 months ago        470MB
training/webapp     latest              6fae60ef3446        5 years ago         349MB
```

## 存出镜像
可以使用docker [image] save命令导出镜像到本地文件。该命令支持-o、-output string参数，导出镜像到指定的文件中。
> docker [image] save

例如，导出本地的ubuntu:18.04镜像为文件ubuntu_18.04.tar：
```
$ docker save -o ubuntu_18.04.tar ubuntu:18.04
```

## 载入镜像
可以使用docker [image] load将导出的tar文件再导入到本地镜像库。支持-i、-input string选项，从指定文件中读入镜像内容。

## 搜寻镜像（仓库）
使用docker search命令可以搜索Docker Hub官方（仓库）中的镜像。
> docker search [option] keyword

例如，从文件ubuntu_18.04.tar导入镜像到本地镜像列表，如下所示：
```
$ docker load -i ubuntu_18.04.tar
```
或者
```
$ docker load < ubuntu_18.04.tar
```

参数：
+ -f, --filter filter：过滤输出内容；
+ --format string：格式化输出内容；
+ --limit int：限制输出结果个数，默认为25个；
+ --no-trunc：不截断输出结果。

## 获取镜像（仓库）
可以使用docker [image] pull命令直接从Docker Hub镜像源来下载镜像。
> docker [image] pull NAME[:TAG]。

其中，NAME是镜像仓库名称（用来区分镜像）, TAG是镜像的标签（往往用来表示版本信息）。
通常情况下，描述一个镜像需要包括“名称+标签”信息。
如果不显式指定TAG，则默认会选择latest标签，这会下载仓库中最新版本的镜像。

参数：
+ -a, --all-tags=true|false：是否获取仓库中的所有镜像，默认为否；
+ --disable-content-trust：取消镜像的内容校验，默认为真。

## 上传镜像（仓库）
可以使用docker [image] push命令上传镜像到仓库，默认上传到Docker Hub官方仓库（需要登录）。
> docker [image] push NAME[:TAG] |[REGISTRY_HOST[:REGISTRY_PORT]/]NAME[:TAG]。

```
$ docker tag test:latest user/test:latest
$ docker push user/test:latest
The push refers to a repository [docker.io/user/test]
Sending image list

Please login prior to push:
Username:
Password:
Email:
```

# 操作Docker容器
容器是Docker的另一个核心概念。简单来说，容器是镜像的一个运行实例。
所不同的是，镜像是静态的只读文件，而容器带有运行时需要的可写文件层，同时，容器中的应用进程处于运行状态。

## 创建容器
> docker [container] create

```
$ docker create -it ubuntu:latest
af8f4f922dafee22c8fe6cd2ae11d16e25087d61f1b1fa55b36e94db7ef45178
```

## 启动容器
> docker [container] start

```
$ docker start af
afl
```

## 新建并启动容器
> docker [container] run
```
$ docker run ubuntu   /bin/echo 'Hello world'
Hello world
```

守护态运行：
```
$ docker run -d ubuntu  /bin/sh -c "while true; do echo hello world; sleep 1; done"
ce554267d7a4c34eefc92c5517051dc37b918b588736d0823e4c846596b04d83
```

## 停止容器


## 查看容器输出
> docker [container] logs

参数：
+ -details：打印详细信息；
+ -f, -follow：持续保持输出；
+ -since string：输出从某个时间开始的日志；
+ -tail string：输出最近的若干日志；
+ -t, -timestamps：显示时间戳信息；
+ -until string：输出某个时间之前的日志。

```
$ docker logs ce554267d7a4
hello world
. . .
```