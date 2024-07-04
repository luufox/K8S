[thwszy.cn](http://thwszy.cn:10087/doc/57/)
# Docker容器的创建--Dockerfile - 容器Docker&k8s

3 min read

---

## 什么是dockerfile?

Dockerfile是一个包含用于组合映像的命令的文本文档。可以使用在命令行中调用任何命令。 Docker通过读取Dockerfile中的指令自动生成映像。

`docker build`命令用于从Dockerfile构建映像。可以在`docker build`命令中使用-f标志指向文件系统中任何位置的Dockerfile。

例如：`docker build -f /path/to/a/Dockerfile`

## Dockerfile的基本结构

Dockerfile 一般分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令，’#’ 为 Dockerfile 中的注释。

## Dockerfile文件说明

Docker以从上到下的顺序运行Dockerfile的指令。为了指定基本映像，第一条指令必须是`FROM`。一个声明以＃字符开头则被视为注释。可以在Docker文件中使用`RUN`，`CMD`，`FROM`，`EXPOSE`，`ENV`等指令。

这里列出了一些常用的指令:  
`FROM`：指定基础镜像，必须为第一个命令

`MAINTAINER`: 维护者信息

`RUN`：构建镜像时执行的命令

`ADD`：将本地文件添加到容器中，tar类型文件会自动解压(网络压缩资源不会被解压)，可以访问网络资源，类似wget

`COPY`：功能类似ADD，但是是不会自动解压文件，也不能访问网络资源

`CMD`：构建容器后调用，也就是在容器启动时才进行调用。

`ENTRYPOINT`：配置容器，使其可执行化。配合CMD可省去"application"，只使用参数。

`LABEL`：用于为镜像添加元数据

`ENV`：设置环境变量

`EXPOSE`：指定于外界交互的端口

`VOLUME`：用于指定持久化目录

`WORKDIR`：工作目录，类似于cd命令

`USER`:指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。使用USER指定用户时，可以使用用户名、UID或GID，或是两者的组合。当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户

`ARG`：用于指定传递给构建运行时的变量

`ONBUILD`：用于设置镜像触发器

## 举个例子

## 构建基础镜像-Cetos7

[https://github.com/CentOS/sig-cloud-instance-images/tree/master/docker](https://github.com/CentOS/sig-cloud-instance-images/tree/master/docker)  
在guthub上下载cetos的文件包

![](http://thwszy.cn:10087/media/202112/2021-12-13_151850_760303.png)

选择需要的版本号下载tar包

![](http://thwszy.cn:10087/media/202112/2021-12-13_152020_817983.png)

下载tar包后和dockerfile放在同一个目录里

编写dockerfile文件

构建docker镜像  
`docker build --rm -t centos:7.8 .`

Export