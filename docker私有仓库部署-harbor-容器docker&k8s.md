[thwszy.cn](http://thwszy.cn:10087/doc/56/)
# Docker私有仓库部署--harbor - 容器Docker&k8s

4 min read

---

## 环境描述

系统版本：Cetos7  
Docker版本：20.10.8

## 环境初始化

## docker安装

见文档：Docker环境部署

## docker-compose安装

```none
#Linux 上我们可以从 Github 上下载它的二进制包来使用，最新发行的版本地址：https://github.com/docker/compose/releases

#运行以下命令以下载 Docker Compose 的当前稳定版本
curl -L "https://github.com/docker/compose/releases/download/2.2.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
#要安装其他版本的 Compose，请替换 2.2.0。

#将可执行权限应用于二进制文件
chmod +x /usr/local/bin/docker-compose

#创建软链接
ln -s /usr/local/bin/docker-compose /usr/sbin/docker-compose
```

#下载不下来可用附件解压即可  
[【附件】docker-compose.zip](http://thwszy.cn:10087/media/attachment/2021/11/docker-compose.zip)

## Harbor部署

## 下载Harbor

Harbor不同版本下载，速度慢：[https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases)

![](http://thwszy.cn:10087/media/202111/2021-11-25_105217_983348.png)

下载最新版本V2.3.4  
offline是离线版本，online是在线安装版本  
如果无法下载可以使用以下附件：  
[【附件】harbor-online-installer-v2.3.4.tgz](http://thwszy.cn:10087/media/attachment/2021/11/harbor-online-installer-v2.3.4.tgz)

## 安装Harbor

```none
#下载并解压
wget https://github.com/goharbor/harbor/releases/download/v2.3.4/harbor-online-installer-v2.3.4.tgz

tar -xvf harbor-online-installer-v2.3.4.tgz 

cd harbor

ll
总用量 32
-rw-r--r-- 1 root root  3361 11月  9 19:03 common.sh
-rw-r--r-- 1 root root  7840 11月  9 19:03 harbor.yml.tmpl
-rwxr-xr-x 1 root root  2500 11月  9 19:03 install.sh
-rw-r--r-- 1 root root 11347 11月  9 19:03 LICENSE
-rwxr-xr-x 1 root root  1881 11月  9 19:03 prepare
```

## 配置Harbor

```none
#拷贝配置文件
cp harbor.yml.tmpl harbor.yml

#修改配置
vim harbor.yml

5 hostname: 192.168.1.34   #设置访问地址，可以是ip、主机名，不可以设置为127.0.0.1或localhost
  7 # http related config
  8 http:             #启用http
  9   # port for http, default is 80. If https enabled, this port will redirect to https port
 10   port: 80       #http默认端口为80
   #将https注释掉，以关闭https支持（根据情况选择使用http还是https，使用https生成相应证书文件即可。）
 12 # https related config
 13 #https:                #注释
 14   # https port for harbor, default is 443
 15  # port: 443           #注释
 16   # The path of cert and key files for nginx
 17  # certificate: /your/certificate/path     /#注释
 18  # private_key: /your/private/key/path     #注释
 ... 
 33 # Remember Change the admin password from UI after launching Harbor.
 34 harbor_admin_password: Harbor12345      #harbor登录密码
 ...
 46 # The default data volume
 47 data_volume: /harbor/data        #修改harbor仓库数据目
...

#创建仓库目录
mkdir -p /harbor/data

#执行官方自带脚本更新配置文件
./prepare
...
Generated and saved secret to file: /data/secret/keys/secretkey
Successfully called func: create_root_cert
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir

#执行harbor安装脚本
./install.sh
...
✔ ----Harbor has been installed and started successfully.----
#看到上面这就算是安装完成了
```

外部访问  
192.168.1.34:80  
![](http://thwszy.cn:10087/media/202111/2021-11-25_141905_001086.png)

账户：admin 密码：Harbor12345（<mark>配置文件里的密码</mark>）  
![](http://thwszy.cn:10087/media/202111/2021-11-25_141825_280388.png)

## 配置SSL

```none
harbor.yaml

# Configuration file of Harbor

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.

#申请的域名
hostname: harbor.thwszy.com

# http related config
#http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
#  port: 80

# https related config
https:
  # https port for harbor, default is 443
  port: 8443
  # The path of cert and key files for nginx
#证书地址
  certificate: /server/harbor/SSL/6710040_harbor.thwszy.com.pem  
#证书地址
  private_key: /server/harbor/SSL/6710040_harbor.thwszy.com.key   
```

## 一些命令

```none
docker-compose up -d （相当于 build + start ）：构建（容器）并启动（容器）整个project的所有service

docker-compose down -v（相当于 stop + rm ）：停止并移除整个project的所有services

docker-compose start [serviceName]：启动已存在但停止的所有service

docker-compose stop [serviceName]：停止运行的service

docker-compose restart [serviceName]: 重启服务

docker-compose images：列出所用的镜像

docker-compose ps：查看正在运行的镜像
```

Export