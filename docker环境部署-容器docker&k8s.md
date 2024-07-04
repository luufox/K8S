[thwszy.cn](http://thwszy.cn:10087/doc/54/)
# Docker环境部署 - 容器Docker&k8s

1 min read

---

## docker相关软件和设置源

`yum install -y yum-utils device-mapper-persistent-data lvm2`

## 使用阿里云的源

`yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`  
**也可以使用其他yum源**

## 查看可用版本

`yum list docker-ce --showduplicate`

## 查询指定版本

`yum list docker-ce.x86_64 --showduplicates | sort -r | grep 版本号`

<mark>版本号替换为需要的版本</mark>

## 安装指定版本

`yum install docker-ce-20.10.8-3.el7.x86_64 docker-ce-cli-20.10.8-3.el7.x86_64 -y`

## 启动 docker

`systemctl start docker`

## 查看状态

`systemctl status docker`

## 设置开机启动

`systemctl enable docker`

## 查看docker版本

`docker version`  
或者  
`docker info`

## 问题解决

`docker info`查看最后两行有错误  
`WARNING: bridge-nf-call-iptables is disabled`  
`WARNING: bridge-nf-call-ip6tables is disabled`

将桥接的IPv4流量传递到iptables的链

#修改docker镜像源

Export