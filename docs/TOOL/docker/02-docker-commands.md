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
以Nvidia 的docker镜像举例：
[参考NVIDIA官网给出的安装教程](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)
```bash
#引入nvidia-docker源：
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

#更新源：
sudo apt update

#安装nvidia-docker，重启docker服务:
sudo apt-get install nvidia-docker2
sudo systemctl restart docker
#镜像载入
sudo docker pull nvidia/cuda:11.2-base nvidia-smi

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



### 镜像加速器

* Docker 中国官方镜像	[https://registry.docker-cn.com](https://registry.docker-cn.com)
* DaoCloud 镜像站	[http://f1361db2.m.daocloud.io](http://f1361db2.m.daocloud.io)
* Azure 中国镜像	[https://dockerhub.azk8s.cn](https://dockerhub.azk8s.cn)
* 科大镜像站	[https://docker.mirrors.ustc.edu.cn](https://docker.mirrors.ustc.edu.cn)
* 阿里云	[https://ud6340vz.mirror.aliyuncs.com](https://ud6340vz.mirror.aliyuncs.com)
* 七牛云	[https://reg-mirror.qiniu.com](https://reg-mirror.qiniu.com)
* 网易云	[https://hub-mirror.c.163.com](https://hub-mirror.c.163.com)
* 腾讯云	[https://mirror.ccs.tencentyun.com](https://mirror.ccs.tencentyun.com)

