[thwszy.cn](http://thwszy.cn:10087/doc/59/)
# Docker的基本命令用法 - 容器Docker&k8s

7 min read

---

## 容器生命周期管理

## run命令

`docker run` ：创建一个新的容器并运行一个命令  
语法：  
`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

OPTIONS说明：

实例

使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。

`docker run --name mynginx -d nginx:latest`

使用镜像nginx:latest以后台模式启动一个容器,并将容器的80端口映射到主机随机端口。

`docker run -P -d nginx:latest`

使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。

`docker run -p 80:80 -v /data:/data -d nginx:latest`

绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上。

`docker run -p 127.0.0.1:80:8080/tcp ubuntu bash`

使用镜像nginx:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。

`runoob@runoob:~$ docker run -it nginx:latest /bin/bash`  
`root@b8573233d675:/#`

## start/stop/restart 命令

`docker start`:启动一个或多个已经被停止的容器

`docker stop` :停止一个运行中的容器

`docker restart` :重启容器  
语法

`docker start [OPTIONS] CONTAINER [CONTAINER...]`

`docker stop [OPTIONS] CONTAINER [CONTAINER...]`

`docker restart [OPTIONS] CONTAINER [CONTAINER...]`

实例

启动已被停止的容器myrunoob

`docker start myrunoob`

停止运行中的容器myrunoob

`docker stop myrunoob`

重启容器myrunoob

`docker restart myrunoob`

## rm 命令

`docker rm` ：删除一个或多个容器。  
语法

`docker rm [OPTIONS] CONTAINER [CONTAINER...]`

OPTIONS说明：

实例

强制删除容器 db01、db02：

`docker rm -f db01 db02`

移除容器 nginx01 对容器 db01 的连接，连接名 db：

`docker rm -l db`

删除容器 nginx01, 并删除容器挂载的数据卷：

`docker rm -v nginx01`

删除所有已经停止的容器：

`docker rm $(docker ps -a -q)`

## exec 命令

`docker exec` ：在运行的容器中执行命令  
语法

`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`

OPTIONS说明：

实例

在容器 mynginx 中以交互模式执行容器内 /root/runoob.sh 脚本:

`runoob@runoob:~$ docker exec -it mynginx /bin/sh /root/runoob.sh`  
`http://www.runoob.com/`

在容器 mynginx 中开启一个交互模式的终端:

`runoob@runoob:~$ docker exec -i -t mynginx /bin/bash`  
`root@b1a0703e41e7:/#`

也可以通过 docker ps -a 命令查看已经在运行的容器，然后使用容器 ID 进入容器。

查看已经在运行的容器 ID：

第一列的 9df70f9a0714 就是容器 ID。

通过 exec 命令对指定的容器执行 bash:

`docker exec -it 9df70f9a0714 /bin/bash`

## 容器操作

## ps 命令

`docker ps` : 列出容器  
语法

`docker ps [OPTIONS]`

OPTIONS说明：

实例

## logs 命令

`docker logs` : 获取容器的日志  
语法

`docker logs [OPTIONS] CONTAINER`

OPTIONS说明：

实例

## cp 命令

`docker cp` :用于容器与主机之间的数据拷贝。  
语法

`docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-`

`docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH`

OPTIONS说明：

实例

将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。

`docker cp /www/runoob 96f7f14e99ab:/www/`

将主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。

`docker cp /www/runoob 96f7f14e99ab:/www`

将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。

`docker cp 96f7f14e99ab:/www /tmp/`

## 镜像仓库

## login/logout 命令

`docker login` : 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

`docker logout` : 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub  
语法

`docker login [OPTIONS] [SERVER]`

`docker logout [OPTIONS] [SERVER]`

OPTIONS说明：

实例

登陆到Docker Hub

`docker login -u 用户名 -p 密码`

登出Docker Hub

`docker logout`

## pull 命令

`docker pull` : 从镜像仓库中拉取或者更新指定镜像  
语法

`docker pull [OPTIONS] NAME[:TAG|@DIGEST]`

OPTIONS说明：

实例

从Docker Hub下载java最新版镜像。

`docker pull java`

从Docker Hub下载REPOSITORY为java的所有镜像。

`docker pull -a java`

## push 命令

`docker push` : 将本地的镜像上传到镜像仓库,要先登陆到镜像仓库  
语法

`docker push [OPTIONS] NAME[:TAG]`

OPTIONS说明：

实例

上传本地镜像myapache:v1到镜像仓库中。

`docker push myapache:v1`

## 本地镜像管理

## images 命令

`docker images` : 列出本地镜像。  
语法

`docker images [OPTIONS] [REPOSITORY[:TAG]]`

OPTIONS说明：

实例

## rmi 命令

`docker rmi` : 删除本地一个或多个镜像。  
语法

`docker rmi [OPTIONS] IMAGE [IMAGE...]`

OPTIONS说明：

实例

## tag 命令

`docker tag` : 标记本地镜像，将其归入某一仓库。  
语法

`docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][/USERNAME]NAME[:TAG]`

实例

## build 命令

`docker build` 命令用于使用 Dockerfile 创建镜像。  
语法

`docker build [OPTIONS] PATH | URL | -`

OPTIONS说明：

实例

使用当前目录的 Dockerfile 创建镜像，标签为 runoob/ubuntu:v1。

`docker build -t runoob/ubuntu:v1 .`

使用URL github.com/creack/docker-firefox 的 Dockerfile 创建镜像。

`docker build github.com/creack/docker-firefox`

也可以通过 -f Dockerfile 文件的位置：

`docker build -f /path/to/a/Dockerfile .`

在 Docker 守护进程执行 Dockerfile 中的指令前，首先会对 Dockerfile 进行语法检查，有语法错误时会返回：

## history 命令

`docker history` : 查看指定镜像的创建历史。  
语法

`docker history [OPTIONS] IMAGE`

OPTIONS说明：

实例

## save 命令

`docker save` : 将指定镜像保存成 tar 归档文件。  
语法

`docker save [OPTIONS] IMAGE [IMAGE...]`

OPTIONS 说明：

实例

## load 命令

`docker load` : 导入使用 docker save 命令导出的镜像。  
语法

`docker load [OPTIONS]`

OPTIONS 说明：

实例

导入镜像：

## info|version

## info 命令

`docker info` : 显示 Docker 系统信息，包括镜像和容器数。。  
语法

`docker info [OPTIONS]`

实例

## version 命令

`docker version` :显示 Docker 版本信息。  
语法

`docker version [OPTIONS]`

OPTIONS说明：

实例

Export