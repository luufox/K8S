[thwszy.cn](http://thwszy.cn:10087/doc/55/)
# k8s环境部署 - 容器Docker&k8s

4 min read

---

## 环境初始化

## 环境描述

系统版本：Cetos7.6  
主机节点：

* k8s-master:192.168.1.31
* k8s-node01:192.168.1.32
* k8s-node02:192.168.1.33

## 关闭selinux和firewalld

```none
#关闭防火墙
[root@localhost ~]# systemctl stop firewalld

#关闭开机自启动防火墙
[root@localhost ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.

#临时关闭selinux
[root@localhost ~]# setenforce 0

#永久关闭selinux
[root@localhost ~]# sed -i 's/enforcing/disabled/g' /etc/selinux/config    #执行完需要重启服务器才生效
```

## 关闭swap

```none
#临时关闭swap
[root@localhost ~]# swapoff -a

#永久关闭swap
[root@localhost ~]# sed -ir 's/.*swap.*/#&/g' /etc/fstab       #需要重启服务器
```

## 设置hostnanme与hosts

```none
#每个节点根据自己习惯去命名
hostnamectl  set-hostname  <hostname>

#每个节点添加hosts节点信息
cat >> /etc/hosts  << EOF
192.168.1.31  k8s-master
192.168.1.32  k8s-node01
192.168.1.33  k8s-node02
EOF
```

---

## k8s环境部署--kubeadm

## 软件版本

* docker 20.10.8
* kubeadm 1.22.2
* kubelet 1.22.2
* kubectl 1.22.2
## docker部署
  详情请查看Docker环境部署

### 修改docker镜像源和调度管理器

```none
vim  /etc/docker/daemon.json
 
{
"registry-mirrors":["https://registry.docker-cn.com","https://hub-mirror.c.163.com","https://docker.mirrors.ustc.edu.cn" ],
"exec-opts":["native.cgroupdriver=systemd"]  # 修改docker的cgroup为systemd
}
```

## 安装相关依赖 （<mark>master操作</mark>）

`yum install conntrack-tools libseccomp libtool-ltdl -y`

## 安装kubeadm、kubelet、kubectl

（<mark>kubectl是master端的管理工具，node节点可以不用安装</mark>）

```none
#由于官方源国内无法访问，这里使用阿里云yum源进行替换：
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

##安装kubeadm、kubelet、kubectl 
yum install -y   kubeadm-1.22.2 kubelet-1.22.2
yum install -y   kubectl-1.22.2  #只需要master安装

#如果安装时提示公钥没有安装，解决方法：--nogpgcheck 进行跳过公钥检查安装

# 设置kubelet开机启动(注意：kubelet服务会暂时启动不了，先不用管它)
systemctl enable kubelet
```

## 在master初始化

```none
kubeadm init \
--apiserver-advertise-address=192.168.1.31 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.22.2 \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16

--apiserver-advertise-address=192.168.1.31 这里填master地址即可

#执行完毕后执行下面得到命令（官方要求不能已root用户运行）
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#为测试方便可以执行下面的命令
mkdir -p ~/.kube
cp -i /etc/kubernetes/admin.conf ~/.kube/config

#记下token（千万不能丢失）
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.31:6443 --token 3mmxoa.tpbik5ua3aqocyuo \
        --discovery-token-ca-cert-hash sha256:94563913f65f7c7e83ce75a9f676c30eb92a8dab40d0c7cbb57654f291c788e9
```

##在node节点加入集群

```none
kubeadm join 192.168.1.31:6443 --token 3mmxoa.tpbik5ua3aqocyuo \
        --discovery-token-ca-cert-hash sha256:94563913f65f7c7e83ce75a9f676c30eb92a8dab40d0c7cbb57654f291c788e9

#加入之后状态仍然不对，那是因为没有安装网络插件
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS     ROLES                  AGE     VERSION
k8s-master   NotReady   control-plane,master   19m     v1.22.2
k8s-node01   NotReady   <none>                 2m43s   v1.22.2
k8s-node02   NotReady   <none> 
```

##安装网络插件

```none
#在每个节点安装插件flannel
mkdir flannel
cd flannel
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
#如果下载不了直接下载附件
[【附件】kube-flannel.yml](/media/attachment/2021/11/kube-flannel.yml)
kubectl apply -f kube-flannel.yml

#有报错不影响使用，别想太多
[root@k8s-node01 flannel]# kubectl apply -f kube-flannel.yml
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

##查询状态

```none
[root@k8s-master flannel]# kubectl get nodes
NAME         STATUS   ROLES                  AGE    VERSION
k8s-master   Ready    control-plane,master   156m   v1.22.2
k8s-node01   Ready    <none>                 140m   v1.22.2
k8s-node02   Ready    <none>                 137m   v1.22.2

[root@k8s-master flannel]# kubectl get  cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
controller-manager   Healthy     ok
etcd-0               Healthy     {"health":"true","reason":""}
# scheduler状态不对，解决方案如下
 # 修改kube-scheduler.yaml文件
   vim /etc/kubernetes/manifests/kube-scheduler.yaml
   将  --port=0 这一行注释

 # 重启kubelet
   systemctl restart kubelet

 # 再次查询状态
 [root@k8s-master ~]# kubectl get  cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true","reason":""}
```

Export