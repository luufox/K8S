[thwszy.cn](http://thwszy.cn:10087/doc/61/)
# Docker的存储目录迁移 - 容器Docker&k8s

6 min read

---

docker的默认存储目录是/var/lib/docker,目录结构如下：

```none
[root@www ~]# cd /var/lib/docker/
[root@www docker]# ll
总用量 16
drwx--x--x  4 root root   120 11月 23 14:07 buildkit
drwx-----x  5 root root   222 12月 20 17:33 containers
drwx------  3 root root    22 11月 23 14:07 image
drwxr-x---  3 root root    19 11月 23 14:07 network
drwx-----x 42 root root 12288 12月 20 17:33 overlay2
drwx------  4 root root    32 11月 23 14:07 plugins
drwx------  2 root root     6 12月  1 16:41 runtimes
drwx------  2 root root     6 11月 23 14:07 swarm
drwx------  2 root root     6 12月 20 16:20 tmp
drwx------  2 root root     6 11月 23 14:07 trust
drwx-----x  2 root root    50 12月  1 16:41 volumes
```

## 查看docker自身的内存占用：

`docker system df`

```none
[root@www docker]# docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          9         3         2.762GB   2.084GB (75%)
Containers      3         0         299.1MB   299.1MB (100%)
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B
```

## 清理无用的docker资源

`docker system prune`  
命令可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)。

`docker system prune -a`  
命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了…所以使用之前一定要想清楚.。我没用过，因为会清理 没有开启的 Docker 镜像。

## 迁移目录

```none
#关闭自动启动docker
systemctl disable docker

#停止docker服务器
 systemctl stop docker
 systemctl stop docker.socket

#创建新的docker目录，找一个大的磁盘。 我在 /server目录
mkdir -p /server/docker

#修改docker.service文件
vim /usr/lib/systemd/system/docker.service

#在里面的EXECStart的后面增加后如下
ExecStart=/usr/bin/dockerd --graph /server/docker

#复制数据
cp -ar /var/lib/docker /server

#重新加载配置
systemctl daemon-reload

#开启自启动
systemctl enable docker

#启动docker
systemctl start docker

#查看是否变更
# Docker Root Dir: /server/docker就是存储目录，已经改变了
[root@www ~]# docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Build with BuildKit (Docker Inc., v0.6.1-docker)
  scan: Docker Scan (Docker Inc., v0.12.0)

Server:
 Containers: 3
  Running: 0
  Paused: 0
  Stopped: 3
 Images: 28
 Server Version: 20.10.8
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 7b11cfaabd73bb80907dd23182b9347b4245eb5d
 runc version: v1.0.2-0-g52b36a2
 init version: de40ad0
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 3.10.0-957.el7.x86_64
 Operating System: CentOS Linux 7 (Core)
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 1.952GiB
 Name: www.harbor.mobi
 ID: EEYB:ZZTI:MQI3:ESRY:3NMG:EB4I:2CAF:DIKG:6B2V:4BWP:JCHH:UDTE
 Docker Root Dir: /server/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  harbor.guoran100.com
  127.0.0.0/8
 Registry Mirrors:
  https://registry.docker-cn.com/
  https://hub-mirror.c.163.com/
  https://docker.mirrors.ustc.edu.cn/
 Live Restore Enabled: false


#删除原有目录数据
rm -rf /var/lib/docker
```

## 问题

今天做这个实验的时候发现一个小问题，为防止以后忘记，记录下来

```none
[root@www ~]# systemctl stop docker
Warning: Stopping docker.service, but it can still be activated by:
  docker.socket

#停止docker的时候出现一个报错，但是看状态也停止了
[root@www ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: inactive (dead) since 二 2021-12-21 13:36:08 CST; 9s ago
     Docs: https://docs.docker.com
  Process: 18557 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --graph /server/docker (code=exited, status=0/SUCCESS)
 Main PID: 18557 (code=exited, status=0/SUCCESS)

12月 21 11:59:56 www.harbor.mobi dockerd[18557]: time="2021-12-21T11:59:56.275387559+08:00" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a ...rred IP address"
12月 21 11:59:56 www.harbor.mobi dockerd[18557]: time="2021-12-21T11:59:56.353483966+08:00" level=info msg="Loading containers: done."
12月 21 11:59:56 www.harbor.mobi dockerd[18557]: time="2021-12-21T11:59:56.369886545+08:00" level=info msg="Docker daemon" commit=75249d8 graphdriver(s)=overlay2 version=20.10.8
12月 21 11:59:56 www.harbor.mobi dockerd[18557]: time="2021-12-21T11:59:56.369947307+08:00" level=info msg="Daemon has completed initialization"
12月 21 11:59:56 www.harbor.mobi systemd[1]: Started Docker Application Container Engine.
12月 21 11:59:56 www.harbor.mobi dockerd[18557]: time="2021-12-21T11:59:56.394219343+08:00" level=info msg="API listen on /var/run/docker.sock"
12月 21 13:36:08 www.harbor.mobi systemd[1]: Stopping Docker Application Container Engine...
12月 21 13:36:08 www.harbor.mobi dockerd[18557]: time="2021-12-21T13:36:08.314822103+08:00" level=info msg="Processing signal 'terminated'"
12月 21 13:36:08 www.harbor.mobi dockerd[18557]: time="2021-12-21T13:36:08.315509192+08:00" level=info msg="Daemon shutdown complete"
12月 21 13:36:08 www.harbor.mobi systemd[1]: Stopped Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.

#实际仍然在运行
[root@www ~]# docker ps -a
CONTAINER ID   IMAGE             COMMAND             CREATED       STATUS                     PORTS     NAMES
1369dbed088f   centos:7.8        "/bin/bash"         2 weeks ago   Exited (0) 2 weeks ago               keen_ellis
d0b9efab42e0   lxid/cetos7:7.5   "/bin/bash"         3 weeks ago   Exited (0) 2 weeks ago               lucid_cori
783d946dca9b   tomcat:8.5.73     "catalina.sh run"   3 weeks ago   Exited (137) 3 weeks ago             admiring_austin

#想要彻底停止，只能运行下面命令
[root@www ~]# systemctl stop docker.socket
[root@www ~]# docker ps -a
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

解释：  
这是因为除了docker.service单元文件，还有一个docker.socket单元文件…docker.socket这是用于套接字激活。  
该警告意味着：如果你试图连接到docker socket，而docker服务没有运行，系统将自动启动docker。

总结  
其实这是个挺人性化的设计，知道意思后，就不想采取什么干预了。

Export