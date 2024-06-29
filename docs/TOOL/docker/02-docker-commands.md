# docker基本命令

参考[https://docs.docker.com/reference/cli/docker/](https://docs.docker.com/reference/cli/docker/)

### docker简介
Docker 是一个应用打包、分发、部署的工具
>打包：把软件运行所需的依赖、第三方库、软件打包到一起，变成一个安装包
分发：把打包好的“安装包”上传到一个镜像仓库，其他人可以非常方便的获取和安装
部署：拿着“安装包”就可以一个命令运行起来你的应用，自动模拟出一摸一样的运行环境，不管是在 Windows/Mac/Linux

确保了不同机器上都是一致的运行环境

镜像：镜像包含运行应用程序所需的所有内容——代码或二进制文件、运行时、依赖项以及所需的任何其他文件系统对象。可以理解为软件安装包，可以方便的进行传播和安装。
容器：容器只不过是一个正在运行的进程，它还应用了一些附加的封装功能，以使其与主机和其他容器保持隔离。容器隔离的最重要方面之一是每个容器都与自己的私有文件系统进行交互；该文件系统由Docker镜像提供。

### 获取镜像
获取镜像的方式有两种： 
* 使用他人打包好，并通过网络（主要是docker官方的docker hub和一些类似的镜像托管网站）进行分享的镜像；
* 在本地将镜像保存为本地文件，直接使用生成的文件进行共享。后者在网络首先环境下更加方便；

##### 从网络
这里主要可以借助于两条指令。一个是 `docker pull` ，另一个是 `docker run `
```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```
顾名思义，就是从仓库中拉取（pull）指定的镜像到本机。看它的配置项，对于可选项我们暂不需要管，我们重点关注后面的 NAME 和紧跟的两个互斥的配置项。
对应于这里的三种构造指令的方式，文档中给出了几种不同的拉取方式：

* NAME
  * `docker pull ubuntu`
如果不指定标签，Docker Engine会使用 :latest 作为默认标签拉取镜像。
* NAME:TAG
  * `docker pull ubuntu:14.04`
使用标签时，可以再次 `docker pull` 这个镜像，以确保具有该镜像的最新版本。
例如，`docker pull ubuntu:14.04` 将会拉取`Ubuntu 14.04`镜像的最新版本。
* NAME@DIGEST
  * `docker pull ubuntu@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`
这个为我们提供了一种指定特定版本镜像的方法
为了保证后期我们仅仅使用这个版本的镜像，我们可以重新通过指定DIGEST（通过查看镜像托管网站里的镜像信息或者是之前的 pull 输出里的DIGEST信息）的方式 pull 该版本镜像。


```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

`docker run` 命令首先在指定的映像上创建一个可写的容器层，然后使用指定的命令启动它。

这个实际上会自动从官方仓库中下载本地没有的镜像。更多是使用会在后面创建容器的部分介绍。

```bash
$ sudo docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE

$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:49a1c8800c94df04e9658809b006fd8a686cab8028d33cfba2cc049724254202
Status: Downloaded newer image for hello-world:latest

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

$ sudo docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
hello-world                                            latest              bf756fb1ae65        7 months ago        13.3kB

```

##### 从他人处
关于如何保存镜像在后面介绍，其涉及到的指令为 `docker save` ，这里主要讲如何加载已经导出的镜像文件。
```bash
docker load [OPTIONS]
```
例子可见：[https://docs.docker.com/engine/reference/commandline/load/#examples](https://docs.docker.com/engine/reference/commandline/load/#examples)

##### 如何使用镜像
单纯的使用镜像实际上就是围绕指令 `docker run` 来的。

首先我们通过使用 `docker images` 来查看本机中已经保存的镜像信息列表。
```bash
$ sudo docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
```


通过运行 `docker run` 获取镜像并创建容器。
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
run包含很多的参数和配置项，这里以nvidia-docker为例[https://github.com/NVIDIA/nvidia-docker#usage](https://github.com/NVIDIA/nvidia-docker#usage)：
```bash
sudo docker run --rm -it --gpus all --name test -v /home/mydataroot:/tcdata:ro nvidia/cuda:10.0-base /bin/bash
```
这条指令做的就是：

启动Docker容器时，必须首先确定是要在后台以 “分离” 模式还是在默认前台模式下运行容器： `-d `，这里没有指定 `-d` 则是使用默认前台模式运行。两种模式下，部分参数配置不同，这部分细节可以参考文档：https://docs.docker.com/engine/reference/run/#detached-vs-foreground
使用镜像 `nvidia/cuda:10.0-base` 创建容器，并对容器起一个别名 `test `。
对于该容器，开启gpu支持，并且所有GPU都可用，但是前提你得装好`nvidia-docker`。
`--rm` 表示退出容器的时候自动移除容器，在测试环境等场景下很方便，不用再手动删除已经创建的容器了。
`-t` 和 `-i` ：这两个参数的作用是，为该docker创建一个伪终端，这样就可以进入到容器的交互模式。
后面的 `/bin/bash` 的作用是表示载入容器后运行 `bash` 。docker中必须要保持一个进程的运行，要不然整个容器启动后就会马上`kill itself`，这样当你使用 `docker ps` 查看启动的容器时，就会发现你刚刚创建的那个容器并不在已启动的容器队列中。 这个 `/bin/bash` 就表示启动容器后启动 `bash` 。
`-v `表示将本地的文件夹以只读`:ro `，读写可以写为 `:rw` ，如果不加，则默认的方式是读写的方式挂载到容器中的 /tcdata 目录中。
```bash
(base) xujg@xujg-ASUS:~/文档/xjg-docs$ sudo docker run -it ubuntu bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
7b1a6ab2e44d: Pull complete 
Digest: sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
Status: Downloaded newer image for ubuntu:latest
root@bf26f5984d7f:/# exit
exit
(base) xujg@xujg-ASUS:~/文档/xjg-docs$ sudo docker images
REPOSITORY    TAG         IMAGE ID       CREATED       SIZE
ubuntu        latest      ba6acccedd29   2 years ago   72.8MB
hello-world   latest      feb5d9fea6a5   2 years ago   13.3kB
nvidia/cuda   10.0-base   97cca2bac989   2 years ago   109MB
(base) xujg@xujg-ASUS:~/文档/xjg-docs$ sudo docker ps # 查看正在运行的容器信息
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
(base) xujg@xujg-ASUS:~/文档/xjg-docs$ sudo docker ps -a # 查看所有容器信息
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
bf26f5984d7f   ubuntu        "bash"     51 seconds ago   Exited (0) 23 seconds ago             pensive_archimedes
048496d67740   hello-world   "/hello"   27 minutes ago   Exited (0) 27 minutes ago             angry_dhawan

```
当我们想要删除指定的镜像的时候，我们可以使用 `sudo docker rmi` 来进行处理。

`docker rmi [OPTIONS] IMAGE [IMAGE...]`
后面可以跟多个镜像。这里支持三种方式：
* 使用IMAGE ID：`docker rmi fd484f19954f`
* 使用TAG： `docker rmi test:latest`
* 使用DIGEST： `docker rmi localhost:5000/test/busybox@sha256:cbbf2f9a99b47fc460d422812b6a5adff7dfee951d8fa2e4a98caa0382cfbdbf`
#### 如何使用容器
##### 创建容器
从前面可以了解到，我们可以通过使用 docker run 创建前台运行的容器，创建好了容器我们会面临如何使用的问题。

##### 进入容器
进入容器的方法都是一致的，但是这里会面临两种情况，一种是已经退出的容器，另一种是运行在后台的分离模式下的容器。

##### 已退出的容器
```bash
$ sudo docker run -it --gpus all --name testv3 -v /home/lart/Downloads/:/data nvidia/cuda:10.0-base bash
root@efd722f0321f:/# exit
exit
(pt16) lart@god:~/Coding/RGBSOD_MS$ sudo docker ps -a
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                         PORTS               NAMES
efd722f0321f        nvidia/cuda:10.0-base   "bash"              19 seconds ago      Exited (0) 3 seconds ago                           testv3
```

可以看到，我这里存在一个已经退出的容器。我们想要进入退出的容器首先需要启动已经退出的容器：
```bash
(pt16) lart@god:~/Coding/RGBSOD_MS$ sudo docker start testv3
testv3
(pt16) lart@god:~/Coding/RGBSOD_MS$ sudo docker ps -a
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                         PORTS               NAMES
efd722f0321f        nvidia/cuda:10.0-base   "bash"              31 seconds ago      Up 1 second    

```
关于重启容器，使用 `docker restart` 也是可以的。为了验证，我们先使用 `docker stop` 停止指定容器。再进行测试

```bash
$ sudo docker stop testv3
testv3
$ sudo docker ps -a
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                         PORTS               NAMES
efd722f0321f        nvidia/cuda:10.0-base   "bash"              2 minutes ago       Exited (0) 4 seconds ago                           testv3
$ sudo docker restart testv3  # restart 不仅可以重启关掉的容器，也可以重启运行中的容器
testv3
$ sudo docker ps -a
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                         PORTS               NAMES
efd722f0321f        nvidia/cuda:10.0-base   "bash"              2 minutes ago       Up 1 second 
```
##### 后台分离模式运行的容器
对于创建分离模式的容器我们可以使用 `docker run -d` 。
另外前面提到的 `start` 或者 `restart` 启动的容器会自动以分离模式运行。

##### 进入启动的容器
对于已经启动的容器，我们可以使用 `attach` 或者 `exec` 进入该容器，但是更推荐后者，区别是`exec`退出容器时，自动关闭

```bash
$ sudo docker ps -a
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                         PORTS               NAMES
efd722f0321f        nvidia/cuda:10.0-base   "bash"              16 minutes ago      Exited (0) 2 minutes ago                           testv3
$ sudo docker start testv3
testv3
$ sudo docker ps -a
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                         PORTS               NAMES
efd722f0321f        nvidia/cuda:10.0-base   "bash"              16 minutes ago      Up 2 seconds                                       testv3

$ sudo docker exec -it testv3 bash
root@efd722f0321f:/# exit
exit
$ sudo docker ps -a # 可以看到exec退出后不会把容器关闭
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                         PORTS               NAMES
efd722f0321f        nvidia/cuda:10.0-base   "bash"              17 minutes ago      Up 36 seconds                                      testv3

$ sudo docker attach testv3
root@efd722f0321f:/# exit
exit
$ sudo docker ps -a  # 可以看到attach退出后会把容器关闭
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                         PORTS               NAMES
efd722f0321f        nvidia/cuda:10.0-base   "bash"              18 minutes ago      Exited (0) 2 seconds ago                           testv3
```
##### 删除容器
```bash
sudo docker rm [OPTIONS] CONTAINER [CONTAINER...]
```
##### 停止正在运行的容器

`docker stop` ：此方式常常被翻译为优雅的停止容器。
`docker stop` 容器ID或容器名
`-t` ：关闭容器的限时，如果超时未能关闭则用 kill 强制关闭，默认值10s，这个时间用于容器的自己保存状态， `docker stop -t=60` 容器ID或容器名
`docker kill` ：直接关闭容器
`docker kill` 容器ID或容器名

##### 从本机与容器中互相拷贝数据
`docker cp` ：用于容器与主机之间的数据拷贝。
```bash
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```
The docker cp utility copies the contents of SRC_PATH to the DEST_PATH. You can copy from the container’s file system to the local machine or the reverse, from the local filesystem to the container.
If - is specified for either the SRC_PATH or DEST_PATH, you can also stream a tar archive from STDIN or to STDOUT.
The CONTAINER can be a running or stopped container.
The SRC_PATH or DEST_PATH can be a file or directory.
由于容器内的数据与容器外的数据并不共享，所以如果我们想要向其中拷贝一些数据，可以通过这个指令来进行复制。当然，另一个比较直接的想法就是通过挂载的目录进行文件共享与复制。

##### 如何生成镜像
主要包含两种方式，一种是基于构建文件`Dockerfile`和 `docker build` 的自动构建，一种是基于 `docker commit` 提交对于现有容器的修改之后生成镜像。
```bash
docker build [OPTIONS] PATH | URL | -
```

更多细节可见[https://docs.docker.com/engine/reference/commandline/build/](https://docs.docker.com/engine/reference/commandline/build/)

这里用到了Dockerfile，这些参考资料不错：

你必须知道的Dockerfile：[https://www.cnblogs.com/edisonchou/p/dockerfile_inside_introduction.html](https://www.cnblogs.com/edisonchou/p/dockerfile_inside_introduction.html)
对于已有的Dockerfile文件，我们可以使用如下指令生成镜像：

```bash
# 使用'.'目录下的Dockerfile文件。注意结尾的路径`.`，这里给打包的镜像指定了TAG
$ docker build -t vieux/apache:2.0 .
# 也可以指定多个TAG
$ docker build -t whenry/fedora-jboss:latest -t whenry/fedora-jboss:v2.1 .
# 也可以不使用'.'目录下的Dockerfile文件，而是使用-f指定文件
$ docker build -f dockerfiles/Dockerfile.debug -t myapp_debug .
```
打包完之后，我们就可以在 `docker images` 中看到新增的镜像了。

另一种方式，通过`docker commit`
```bash
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```
比如这样
```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS              NAMES
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            focused_hamilton
$ docker commit c3f279d17e0a  svendowideit/testimage:version3
f5283438590d
$ docker images
REPOSITORY                        TAG                 ID                  CREATED             SIZE
svendowideit/testimage            version3            f5283438590d        16 seconds ago      335.7 MB
```

##### 如何分享镜像
###### 上传到在线存储库
上传到Docker Hub：[https://docs.docker.com/get-started/part3/](https://docs.docker.com/get-started/part3/)
上传到阿里云：[https://blog.csdn.net/xiayto/article/details/104133417/](https://blog.csdn.net/xiayto/article/details/104133417/)
###### 本地导出分享
这里基于指令 `docker save`

`docker save [OPTIONS] IMAGE [IMAGE...]`
Produces a tarred repository to the standard output stream. Contains all parent layers, and all tags + versions, or specified repo:tag , for each argument provided.

具体的例子可见：[https://docs.docker.com/engine/reference/commandline/save/#examples](https://docs.docker.com/engine/reference/commandline/save/#examples)

```bash
$ sudo docker ps -a
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                         PORTS               NAMES
153999a1dfb2        hello-world             "/hello"            2 hours ago         Exited (0) 2 hours ago                             naughty_euler
$ sudo docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
hello-world                                            latest              bf756fb1ae65        7 months ago        13.3kB
$ sudo docker save -o hello-world-latest.tar hello-world:latest
$ ls
hello-world-latest.tar

$ sudo docker rmi hello-world:latest
Error response from daemon: conflict: unable to remove repository reference "hello-world:latest" (must force) - container 153999a1dfb2 is using its referenced image bf756fb1ae65
$ sudo docker rmi -f hello-world:latest  # 强制删除镜像
Untagged: hello-world:latest
Untagged: hello-world@sha256:49a1c8800c94df04e9658809b006fd8a686cab8028d33cfba2cc049724254202
Deleted: sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b

$ sudo docker ps -a  # 课件，强制删除镜像后，原始关联的容器并不会被删除
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                   PORTS               NAMES
153999a1dfb2        bf756fb1ae65            "/hello"            5 hours ago         Exited (0) 5 hours ago                       naughty_euler
$ sudo docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE

$ sudo docker load -i hello-world-latest.tar 
Loaded image: hello-world:latest

$ sudo docker images
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
hello-world                                            latest              bf756fb1ae65        7 months ago        13.3kB
$ sudo docker ps -a  # 可见，当重新加载对应的镜像时，这里的IMAGE又对应了回来
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                   PORTS               NAMES
153999a1dfb2        hello-world             "/hello"            5 hours ago         Exited (0) 5 hours ago                       naughty_euler
```
这里也有个例子：

docker 打包本地镜像，并到其他机器进行恢复：[https://blog.csdn.net/zf3419/article/details/88533274](https://blog.csdn.net/zf3419/article/details/88533274)
参考资料
docker命令行的基本指令都在这里：[https://docs.docker.com/engine/reference/commandline/docker/](https://docs.docker.com/engine/reference/commandline/docker/)
官方给了一些指导性的文档：[https://docs.docker.com/develop/](https://docs.docker.com/develop/)
