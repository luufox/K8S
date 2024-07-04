[thwszy.cn](http://thwszy.cn:10087/doc/81/)
# Docker打印启动命令--runlike - 容器Docker&k8s

2 min read

---

在工作中，有些容器服务经历维护人员的变换后，年久失修，无法准确快速的查到启动命令，这个时候，可以使用runlike

环境：CentOS7 python2

## 安装runlike

```none
yum -y install epel-release  ##增加epl源

yum install -y python-pip   ##安装pip

pip install runlike  ##安装runlike
```

## 报错解决

* pip install runlike报错
```none
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-BPSMwy/click/
You are using pip version 8.1.2, however version 22.0.4 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```
  解决办法：

```none
wget https://bootstrap.pypa.io/get-pip.py 
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py  ##选择其中之一执行

python get-pip.py

pip install runlike
```

* python get-pip.py报错
```none
[root@10-10-148-23 ~]# python get-pip.py 
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
Collecting pip<21.0
Downloading pip-20.3.4-py2.py3-none-any.whl (1.5 MB)
   |██████████▊                     | 512 kB 3.6 kB/s eta 0:04:40ERROR: Exception:
Traceback (most recent call last):
```
  解决办法：  
  官网地址：[https://pypi.org/project/pip/20.3.4/#history](https://pypi.org/project/pip/20.3.4/#history)  
  下载`pip-20.3.4-py2.py3-none-any.whl`  
  如下图：

![](http://thwszy.cn:10087/media/202208/2022-08-17_140323_5340110.6302098571492505.png)

```none
python -m pip install --upgrade pip-20.3.4-py2.py3-none-any.whl  ##离线安装pip
```

Export