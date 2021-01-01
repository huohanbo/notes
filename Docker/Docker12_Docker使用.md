# 参考资料
> [https://www.runoob.com/docker/docker-tutorial.html](https://www.runoob.com/docker/docker-tutorial.html)
> [https://www.runoob.com/docker/docker-resources.html](https://www.runoob.com/docker/docker-resources.html)

# Docker 基本使用
## 运行一个应用程序
Docker 允许你在容器内运行应用程序， 使用 docker run 命令来在容器内运行一个应用程序。

输出Hello world：
```
$ docker run ubuntu:15.10 /bin/echo "Hello world"
Hello world
```

各个参数解析：
+ docker: Docker 的二进制执行文件。
+ run: 与前面的 docker 组合来运行一个容器。
+ ubuntu:15.10 指定要运行的镜像，Docker 首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。
+ /bin/echo "Hello world": 在启动的容器里执行的命令

以上命令完整的意思可以解释为：
Docker 以 ubuntu15.10 镜像创建一个新容器，然后在容器里执行 bin/echo "Hello world"，然后输出结果。

## 运行交互式的容器
我们通过 docker 的两个参数 -i -t，让 docker 运行的容器实现"对话"的能力：
```
runoob@runoob:~$ docker run -i -t ubuntu:15.10 /bin/bash
root@0123ce188bd8:/#
```

各个参数解析：
+ -t: 在新容器内指定一个伪终端或终端。
+ -i: 允许你对容器内的标准输入 (STDIN) 进行交互。

注意第二行 root@0123ce188bd8:/#，此时我们已进入一个 ubuntu15.10 系统的容器。

我们尝试在容器中运行命令 cat /proc/version和ls分别查看当前系统的版本信息和当前目录下的文件列表
```
root@0123ce188bd8:/#  cat /proc/version
Linux version 4.4.0-151-generic (buildd@lgw01-amd64-043) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10) ) #178-Ubuntu SMP Tue Jun 11 08:30:22 UTC 2019
root@0123ce188bd8:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@0123ce188bd8:/# 
```

我们可以通过运行 exit 命令或者使用 CTRL+D 来退出容器。
```
root@0123ce188bd8:/#  exit
exit
root@runoob:~# 
```

注意第三行中 root@runoob:~# 表明我们已经退出了当期的容器，返回到当前的主机中。

## 运行后台进程式的容器
使用以下命令创建一个以进程方式运行的容器
```
runoob@runoob:~$ docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
2b1b7a428627c51ab8810d541d759f072b4fc75487eed05812646b8534a2fe63
```

在输出中，我们没有看到期望的 "hello world"，而是一串长字符2b1b7a428627c51ab8810d541d759f072b4fc75487eed05812646b8534a2fe63。
这个长字符串叫做容器 ID，对每个容器来说都是唯一的，我们可以通过容器 ID 来查看对应的容器发生了什么。

首先，我们需要确认容器有在运行，可以通过 docker ps 来查看：
```
runoob@runoob:~$ docker ps
CONTAINER ID        IMAGE                  COMMAND              ...  
5917eac21c36        ubuntu:15.10           "/bin/sh -c 'while t…"    ...
```

输出详情介绍：
+ CONTAINER ID: 容器 ID。
+ IMAGE: 使用的镜像。
+ COMMAND: 启动容器时运行的命令。
+ CREATED: 容器的创建时间。
+ STATUS: 容器状态。
+ PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。
+ NAMES: 自动分配的容器名称。

状态有7种：
+ created（已创建）
+ restarting（重启中）
+ running（运行中）
+ removing（迁移中）
+ paused（暂停）
+ exited（停止）
+ dead（死亡）

在宿主主机内使用 docker logs 命令，查看容器内的标准输出：
```
runoob@runoob:~$ docker logs 2b1b7a428627
```
或
```
runoob@runoob:~$ docker logs amazing_cori
```

## 停止容器
我们使用 docker stop 命令来停止容器:
```
runoob@runoob:~$ docker stop 2b1b7a428627
```
或
```
runoob@runoob:~$ docker stop amazing_cori
```

# Docker 镜像使用
当运行容器时，使用的镜像如果在本地中不存在，docker 就会自动从 docker 镜像仓库中下载，默认是从 Docker Hub 公共镜像源下载。

## 列出镜像列表
我们可以使用 docker images 来列出本地主机上的镜像。
```
runoob@runoob:~$ docker images           
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               90d5884b1ee0        5 days ago          188 MB
php                 5.6                 f40e9e0f10c8        9 days ago          444.8 MB
nginx               latest              6f8d099c3adc        12 days ago         182.7 MB
mysql               5.6                 f2e8d6c772c0        3 weeks ago         324.6 MB
httpd               latest              02ef73cf1bc0        3 weeks ago         194.4 MB
ubuntu              15.10               4e3b13c8a266        4 weeks ago         136.3 MB
hello-world         latest              690ed74de00f        6 months ago        960 B
training/webapp     latest              6fae60ef3446        11 months ago       348.8 MB
```

各个选项说明:
+ REPOSITORY：表示镜像的仓库源
+ TAG：镜像的标签
+ IMAGE ID：镜像ID
+ CREATED：镜像创建时间
+ SIZE：镜像大小

同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，如 ubuntu 仓库源里，有 15.10、14.04 等多个不同的版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。
所以，我们如果要使用版本为15.10的ubuntu系统镜像来运行容器时，命令如下：
```
runoob@runoob:~$ docker run -t -i ubuntu:15.10 /bin/bash 
root@d77ccb2e5cca:/#
```

参数说明：
+ -i: 交互式操作。
+ -t: 终端。
+ ubuntu:15.10: 这是指用 ubuntu 15.10 版本镜像为基础来启动容器。
+ /bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

如果要使用版本为 14.04 的 ubuntu 系统镜像来运行容器时，命令如下：
```
runoob@runoob:~$ docker run -t -i ubuntu:14.04 /bin/bash 
root@39e968165990:/# 
```

如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像。

## 获取镜像
当我们在本地主机上使用一个不存在的镜像时 Docker 就会自动下载这个镜像。如果我们想预先下载这个镜像，我们可以使用 docker pull 命令来下载它。
```
Crunoob@runoob:~$ docker pull ubuntu:13.10
13.10: Pulling from library/ubuntu
6599cadaf950: Pull complete 
23eda618d451: Pull complete 
f0be3084efe9: Pull complete 
52de432f084b: Pull complete 
a3ed95caeb02: Pull complete 
Digest: sha256:15b79a6654811c8d992ebacdfbd5152fcf3d165e374e264076aa435214a947a3
Status: Downloaded newer image for ubuntu:13.10
```

下载完成后，我们可以直接使用这个镜像来运行容器。

## 查找镜像
我们可以从 Docker Hub 网站来搜索镜像，Docker Hub 网址为： https://hub.docker.com/。

我们也可以使用 docker search 命令来搜索镜像。

比如我们需要一个 httpd 的镜像来作为我们的 web 服务。我们可以通过 docker search 命令搜索 httpd 来寻找适合我们的镜像。
```
runoob@runoob:~$  docker search httpd
NAME                                    DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
httpd                                   The Apache HTTP Server Project                  3050                [OK]
centos/httpd-24-centos7                 Platform for running Apache httpd 2.4 or bui…   32
centos/httpd                                                                            29                                      [OK]
```

参数说明：
+ NAME: 镜像仓库源的名称
+ DESCRIPTION: 镜像的描述
+ OFFICIAL: 是否 docker 官方发布
+ stars: 类似 Github 里面的 star，表示点赞、喜欢的意思。
+ AUTOMATED: 自动构建。

## 删除镜像
镜像删除使用 docker rmi 命令，比如我们删除 hello-world 镜像：
```
$ docker rmi hello-world
```

## 创建镜像
当我们从 docker 镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。
+ 1、从已经创建的容器中更新镜像，并且提交这个镜像
+ 2、使用 Dockerfile 指令来创建一个新的镜像

### 更新镜像
更新镜像之前，我们需要使用镜像来创建一个容器。
```
runoob@runoob:~$ docker run -t -i ubuntu:15.10 /bin/bash
root@e218edb10161:/# 
```

在运行的容器内使用 apt-get update 命令进行更新。在完成操作之后，输入 exit 命令来退出这个容器。

此时 ID 为 e218edb10161 的容器，是按我们的需求更改的容器。我们可以通过命令 docker commit 来提交容器副本。
```
runoob@runoob:~$ docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2
sha256:70bf1840fd7c0d2d8ef0a42a817eb29f854c1af8f7c59fc03ac7bdee9545aff8
```
```
docker commit -m="test commit" -a="huohanbo" aed6d4e2f64b huohanbo/myubuntu:0.0.1
sha256:1c983c3bb4f19b0dca1d1eec458f67c8148fcd4700d31d08de598fce3075f3c3
```

各个参数说明：
+ -m: 提交的描述信息
+ -a: 指定镜像作者
+ e218edb10161：容器 ID
+ runoob/ubuntu:v2: 指定要创建的目标镜像名

我们可以使用 docker images 命令来查看我们的新镜像 runoob/ubuntu:v2：
```
runoob@runoob:~$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
runoob/ubuntu       v2                  70bf1840fd7c        15 seconds ago      158.5 MB
ubuntu              14.04               90d5884b1ee0        5 days ago          188 MB
php                 5.6                 f40e9e0f10c8        9 days ago          444.8 MB
nginx               latest              6f8d099c3adc        12 days ago         182.7 MB
mysql               5.6                 f2e8d6c772c0        3 weeks ago         324.6 MB
httpd               latest              02ef73cf1bc0        3 weeks ago         194.4 MB
ubuntu              15.10               4e3b13c8a266        4 weeks ago         136.3 MB
hello-world         latest              690ed74de00f        6 months ago        960 B
training/webapp     latest              6fae60ef3446        12 months ago       348.8 MB
```

使用我们的新镜像 runoob/ubuntu 来启动一个容器：
```
runoob@runoob:~$ docker run -t -i runoob/ubuntu:v2 /bin/bash                            
root@1a9fbdeb5da3:/#
```

### 构建镜像
我们使用命令 docker build ， 从零开始来创建一个新的镜像。
为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。
```
runoob@runoob:~$ cat Dockerfile 
FROM    centos:6.7
MAINTAINER      Fisher "fisher@sudops.com"
RUN     /bin/echo 'root:123456' |chpasswd
RUN     useradd runoob
RUN     /bin/echo 'runoob:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
```

**每一个指令都会在镜像上创建一个新的层**，每一个指令的前缀都必须是大写的。
第一条FROM，指定使用哪个镜像源。
RUN 指令告诉docker 在镜像内执行命令，安装了什么。

然后，我们使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。
```
runoob@runoob:~$ docker build -t runoob/centos:6.7 .
Sending build context to Docker daemon 17.92 kB
Step 1 : FROM centos:6.7
 ---&gt; d95b5ca17cc3
Step 2 : MAINTAINER Fisher "fisher@sudops.com"
 ---&gt; Using cache
 ---&gt; 0c92299c6f03
Step 3 : RUN /bin/echo 'root:123456' |chpasswd
 ---&gt; Using cache
 ---&gt; 0397ce2fbd0a
Step 4 : RUN useradd runoob
......
```

参数说明：
+ -t ：指定要创建的目标镜像名
+ . ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径

使用docker images 查看创建的镜像已经在列表中存在,镜像ID为860c279d2fec
```
runoob@runoob:~$ docker images 
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
runoob/centos       6.7                 860c279d2fec        About a minute ago   190.6 MB
runoob/ubuntu       v2                  70bf1840fd7c        17 hours ago         158.5 MB
ubuntu              14.04               90d5884b1ee0        6 days ago           188 MB
php                 5.6                 f40e9e0f10c8        10 days ago          444.8 MB
nginx               latest              6f8d099c3adc        12 days ago          182.7 MB
mysql               5.6                 f2e8d6c772c0        3 weeks ago          324.6 MB
httpd               latest              02ef73cf1bc0        3 weeks ago          194.4 MB
ubuntu              15.10               4e3b13c8a266        5 weeks ago          136.3 MB
hello-world         latest              690ed74de00f        6 months ago         960 B
centos              6.7                 d95b5ca17cc3        6 months ago         190.6 MB
training/webapp     latest              6fae60ef3446        12 months ago        348.8 MB
```

我们可以使用新的镜像来创建容器
```
runoob@runoob:~$ docker run -t -i runoob/centos:6.7  /bin/bash
[root@41c28d18b5fb /]# id runoob
uid=500(runoob) gid=500(runoob) groups=500(runoob)
```

从上面看到新镜像已经包含我们创建的用户 runoob。

## 设置镜像标签
我们可以使用 docker tag 命令，为镜像添加一个新的标签。
```
runoob@runoob:~$ docker tag 860c279d2fec runoob/centos:dev
```

使用 docker images 命令可以看到，ID为860c279d2fec的镜像多一个标签。
```
runoob@runoob:~$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
runoob/centos       6.7                 860c279d2fec        5 hours ago         190.6 MB
runoob/centos       dev                 860c279d2fec        5 hours ago         190.6 MB
runoob/ubuntu       v2                  70bf1840fd7c        22 hours ago        158.5 MB
```

# Docker 仓库使用
仓库（Repository）是集中存放镜像的地方。以下介绍一下 Docker Hub。当然不止 docker hub，只是远程的服务商不一样，操作都是一样的。

## Docker Hub
目前 Docker 官方维护了一个公共仓库 [Docker Hub](https://hub.docker.com/)。

大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。

## 注册
在 https://hub.docker.com 免费注册一个 Docker 账号。

## 登录和退出
登录需要输入用户名和密码，登录成功后，我们就可以从 docker hub 上拉取自己账号下的全部镜像。
```
$ docker login
```

退出 docker hub 可以使用以下命令：
```
$ docker logout
```

## 拉取镜像
以 ubuntu 为关键词进行搜索：
```
$ docker search ubuntu
```

使用 docker pull 将官方 ubuntu 镜像下载到本地：
```
$ docker pull ubuntu 
```

## 推送镜像
用户登录后，可以通过 docker push 命令将自己的镜像推送到 Docker Hub。

以下命令中的 username 请替换为你的 Docker 账号用户名。
```
$ docker tag ubuntu:18.04 username/ubuntu:18.04
$ docker image ls

REPOSITORY      TAG        IMAGE ID            CREATED           ...  
ubuntu          18.04      275d79972a86        6 days ago        ...  
username/ubuntu 18.04      275d79972a86        6 days ago        ...  
$ docker push username/ubuntu:18.04
$ docker search username/ubuntu

NAME             DESCRIPTION       STARS         OFFICIAL    AUTOMATED
username/ubuntu
```

# Docker 容器使用
## 启动一个新的容器
以下命令使用 ubuntu 镜像启动一个容器，参数为以命令行模式进入该容器：
```
$ docker run -it ubuntu /bin/bash
```
+ -i: 交互式操作。
+ -t: 终端。

## 启动已停止的容器
```
$ docker start b750bbbcfd88
```

## 后台启动容器
在大部分的场景下，我们希望 docker 的服务是在后台运行的，我们可以过 -d 指定容器的运行模式。
```
$ docker run -itd --name ubuntu-test ubuntu /bin/bash
```

## 停止容器
```
$ docker stop <容器 ID>
```

## 重启容器
```
$ docker restart <容器 ID>
```

## 进入(后台)容器
在使用 -d 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：
+ docker attach
+ docker exec：推荐大家使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。
```
$ docker attach 1e560fca3906 
```

```
docker exec -it 243c32535da7 /bin/bash
```

## 导出容器
导出容器 1e560fca3906 快照到本地文件 ubuntu.tar。
```
$ docker export 1e560fca3906 > ubuntu.tar
```

## 导入容器
```
$ cat docker/ubuntu.tar | docker import - test/ubuntu:v1
```

## 删除容器
删除容器使用 docker rm 命令：
```
$ docker rm -f 1e560fca3906
```

下面的命令可以清理掉所有处于终止状态的容器：
```
$ docker container prune
```

## 运行一个 web 应用
前面我们运行的容器并没有一些什么特别的用处。
接下来让我们尝试使用 docker 构建一个 web 应用程序。

我们将在docker容器中运行一个 Python Flask 应用来运行一个web应用。
```
runoob@runoob:~# docker pull training/webapp  # 载入镜像
runoob@runoob:~# docker run -d -P training/webapp python app.py
```

参数说明:
+ -d:让容器在后台运行。
+ -P:将容器内部使用的网络端口映射到我们使用的主机上。

## 查看 web 应用容器
使用 docker ps 来查看我们正在运行的容器：
```
runoob@runoob:~#  docker ps
CONTAINER ID        IMAGE               COMMAND             ...        PORTS                 
d3d5e39ed9d3        training/webapp     "python app.py"     ...        0.0.0.0:32769->5000/tcp
```

这里多了端口信息。
```
PORTS
0.0.0.0:32769->5000/tcp
```

Docker 开放了 5000 端口（默认 Python Flask 端口）映射到主机端口 32769 上。

这时我们可以通过浏览器访问WEB应用：[http://127.0.0.1:32768/](http://127.0.0.1:32768/)。

我们也可以通过 -p 参数来设置不一样的端口：
```
runoob@runoob:~$ docker run -d -p 5000:5000 training/webapp python app.py
```

docker ps查看正在运行的容器
```
runoob@runoob:~#  docker ps
CONTAINER ID        IMAGE                             PORTS                     NAMES
bf08b7f2cd89        training/webapp     ...        0.0.0.0:5000->5000/tcp    wizardly_chandrasekhar
d3d5e39ed9d3        training/webapp     ...        0.0.0.0:32769->5000/tcp   xenodochial_hoov
```

容器内部的 5000 端口映射到我们本地主机的 5000 端口上。

## 网络端口的快捷方式
通过 docker ps 命令可以查看到容器的端口映射，docker 还提供了另一个快捷方式 docker port，使用 docker port 可以查看指定 （ID 或者名字）容器的某个确定端口映射到宿主机的端口号。

上面我们创建的 web 应用容器 ID 为 bf08b7f2cd89 名字为 wizardly_chandrasekhar。

我可以使用 docker port bf08b7f2cd89 或 docker port wizardly_chandrasekhar 来查看容器端口的映射情况。
```
runoob@runoob:~$ docker port bf08b7f2cd89
5000/tcp -> 0.0.0.0:5000
runoob@runoob:~$ docker port wizardly_chandrasekhar
5000/tcp -> 0.0.0.0:5000
```

## 查看 WEB 应用程序日志
docker logs [ID或者名字] 可以查看容器内部的标准输出。
```
runoob@runoob:~$ docker logs -f bf08b7f2cd89
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
192.168.239.1 - - [09/May/2016 16:30:37] "GET / HTTP/1.1" 200 -
192.168.239.1 - - [09/May/2016 16:30:37] "GET /favicon.ico HTTP/1.1" 404 -
```

-f: 让 docker logs 像使用 tail -f 一样来输出容器内部的标准输出。

从上面，我们可以看到应用程序使用的是 5000 端口并且能够查看到应用程序的访问日志。

## 查看WEB应用程序容器的进程
我们还可以使用 docker top 来查看容器内部运行的进程
```
runoob@runoob:~$ docker top wizardly_chandrasekhar
UID     PID         PPID          ...       TIME                CMD
root    23245       23228         ...       00:00:00            python app.py
```

## 检查 WEB 应用程序
使用 docker inspect 来查看 Docker 的底层信息。它会返回一个 JSON 文件记录着 Docker 容器的配置和状态信息。
```
runoob@runoob:~$ docker inspect wizardly_chandrasekhar
```

## 停止 WEB 应用容器
```
runoob@runoob:~$ docker stop wizardly_chandrasekhar   
wizardly_chandrasekhar
```

## 重启WEB应用容器
已经停止的容器，我们可以使用命令 docker start 来启动。
```
runoob@runoob:~$ docker start wizardly_chandrasekhar
wizardly_chandrasekhar
```

## 移除WEB应用容器
我们可以使用 docker rm 命令来删除不需要的容器
```
runoob@runoob:~$ docker rm wizardly_chandrasekhar  
wizardly_chandrasekhar
```

# Docker 容器连接
前面我们实现了通过网络端口来访问运行在 docker 容器内的服务。
容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 -P 或 -p 参数来指定端口映射。

## 网络端口映射
使用 -P 参数创建一个容器，使用 docker ps 可以看到容器端口 5000 绑定主机端口 32768。
```
runoob@runoob:~$ docker run -d -P training/webapp python app.py
fce072cc88cee71b1cdceb57c2821d054a4a59f67da6b416fceb5593f059fc6d

```

也可以使用 -p 标识来指定容器端口绑定到主机端口。
```
runoob@runoob:~$ docker run -d -p 5000:5000 training/webapp python app.py
33e4523d30aaf0258915c368e66e03b49535de0ef20317d3f639d40222ba6bc0
```

可以指定容器绑定的网络地址，比如绑定 127.0.0.1。
```
runoob@runoob:~$ docker run -d -p 127.0.0.1:5001:5000 training/webapp python app.py
95c6ceef88ca3e71eaf303c2833fd6701d8d1b2572b5613b5a932dfdfe8a857c
```

默认都是绑定 tcp 端口，如果要绑定 UDP 端口，可以在端口后面加上 /udp。
```
runoob@runoob:~$ docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
6779686f06f6204579c1d655dd8b2b31e8e809b245a97b2d3a8e35abe9dcd22a
```

## Docker 容器互联
端口映射并不是唯一把 docker 连接到另一个容器的方法。
docker 有一个连接系统允许将多个容器连接在一起，共享连接信息。

docker 连接会创建一个父子关系，其中父容器可以看到子容器的信息。

### 容器命名
当我们创建一个容器的时候，docker 会自动对它进行命名。另外，我们也可以使用 --name 标识来命名容器，例如：
```
runoob@runoob:~$  docker run -d -P --name runoob training/webapp python app.py
43780a6eabaaf14e590b6e849235c75f3012995403f97749775e38436db9a441
```

### 新建网络
下面先创建一个新的 Docker 网络。
```
$ docker network create -d bridge test-net
```
参数说明：
+ -d：参数指定 Docker 网络类型，有 bridge、overlay。

### 连接容器
运行一个容器并连接到新建的 test-net 网络:
```
$ docker run -itd --name test1 --network test-net ubuntu /bin/bash
```

打开新的终端，再运行一个容器并加入到 test-net 网络:
```
$ docker run -itd --name test2 --network test-net ubuntu /bin/bash
```

# Docker Dockerfile
## 什么是 Dockerfile？
Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

## 执行构建
在 Dockerfile 文件的存zhil放目录下，执行构建动作：
```
$ docker build -t nginx:test .
```
注：最后的 . 代表本次执行的上下文路径。

## 上下文路径
上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。

注意：上下文路径下不要放无用的文件，因为会一起打包发送给 docker 引擎，如果文件过多会造成过程缓慢。

## 指令详解
### FROM
定制的镜像都是基于 FROM 的镜像，后续的操作都是基于 FROM 的镜像。

### COPY
复制指令，从上下文目录中复制文件或者目录到容器里指定路径。

格式：
```
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
[--chown=<user>:<group>]：可选参数，用户改变复制到容器内文件的拥有者和属组。
```

<源路径>：源文件或者源目录，这里可以是通配符表达式，其通配符规则要满足 Go 的 filepath.Match 规则。例如：
```
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

<目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

### ADD
ADD 指令和 COPY 的使用格式一致（同样需求下，官方推荐使用 COPY）。功能也类似，不同之处如下：
+ ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>。
+ ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定。

### RUN
用于执行后面跟着的命令行命令。有以下俩种格式：

shell 格式：
```
RUN <命令行命令>
<命令行命令> 等同于，在终端操作的 shell 命令。
```

exec 格式：
```
RUN ["可执行文件", "参数1", "参数2"]
RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```

注意：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。例如：
```
FROM centos
RUN yum install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz
```
以上执行会创建 3 层镜像。可简化为以下格式：
```
FROM centos
RUN yum install wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && tar -xvf redis.tar.gz
```
如上，以 && 符号连接命令，这样执行后，只会创建 1 层镜像。

### CMD
为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。

CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。

注意：如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。

类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:
+ CMD 在docker run 时运行。
+ RUN 是在 docker build。

格式：
```
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```
推荐使用第二种格式，执行过程比较明确。第一种格式实际上在运行的过程中也会自动转换成第二种格式运行，并且默认可执行文件是 sh。

### ENTRYPOINT
类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。
但是, 如果运行 docker run 时使用了 --entrypoint 选项，此选项的参数可当作要运行的程序覆盖 ENTRYPOINT 指令指定的程序。

优点：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。

注意：如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。

格式：
```
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参，以下示例会提到。

示例：
假设已通过 Dockerfile 构建了 nginx:test 镜像：
```
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```

### ENV
设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。

格式：
```
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

以下示例设置 NODE_VERSION = 7.2.0 ， 在后续的指令中可以通过 $NODE_VERSION 引用：
```
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
```

### ARG
构建参数，与 ENV 作用一至。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。

构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。

格式：
```
ARG <参数名>[=<默认值>]
```

### VOLUME
定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

作用：
+ 避免重要的数据，因容器重启而丢失，这是非常致命的。
+ 避免容器不断变大。

格式：
```
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。

### EXPOSE
仅仅只是声明端口。

作用：
+ 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
+ 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

格式：
```
EXPOSE <端口1> [<端口2>...]
```

### WORKDIR
指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

格式：
```
WORKDIR <工作目录路径>
```

### USER
用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。

格式：
```
USER <用户名>[:<用户组>]
```

### HEALTHCHECK
用于指定某个程序或者指令来监控 docker 容器服务的运行状态。

格式：
```
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。
```

### ONBUILD
用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这是执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

格式：
```
ONBUILD <其它指令>
```

# Docker Compose

# Docker Machine

# Swarm 集群管理