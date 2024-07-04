[thwszy.cn](http://thwszy.cn:10087/doc/62/)
# docker-compose教程(安装和使用） - 容器Docker&k8s

3 min read

---

## Docker-Compose介绍

Docker Compose是一个用来定义和运行复杂应用的Docker工具。一个使用Docker容器的应用，通常由多个容器组成。使用Docker Compose不再需要使用shell脚本来启动容器。  
Compose 通过一个配置文件来管理多个Docker容器，在配置文件中，所有的容器通过services来定义，然后使用docker-compose脚本来启动，停止和重启应用，和应用中的服务以及所有依赖服务的容器，非常适合组合使用多个容器进行开发的场景。

## 下载docker-compose

[【附件】docker-compose.zip](http://thwszy.cn:10087/media/attachment/2021/12/docker-compose.zip)

## docker-compose命令

docker-compose命令的基本使用格式是：  
`docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]`

常用命令：

* docker-compose 命令 --help 获得一个命令的帮助
* docker-compose up -d nginx 构建启动nignx容器
* docker-compose exec nginx bash 登录到nginx容器中
* docker-compose down 此命令将会停止 up 命令所启动的容器，并移除网络
* docker-compose ps 列出项目中目前的所有容器
* docker-compose restart nginx 重新启动nginx容器
* docker-compose build nginx 构建镜像
* docker-compose build --no-cache nginx 不带缓存的构建
* docker-compose top 查看各个服务容器内运行的进程
* docker-compose logs -f nginx 查看nginx的实时日志
* docker-compose images 列出 Compose 文件包含的镜像
* docker-compose config 验证文件配置，当配置正确时，不输出任何内容，当文件配置错误，输出错误信息。
* docker-compose events --json nginx 以json的形式输出nginx的docker日志
* docker-compose pause nginx 暂停nignx容器
* docker-compose unpause nginx 恢复ningx容器
* docker-compose rm nginx 删除容器（删除前必须关闭容器，执行stop）
* docker-compose stop nginx 停止nignx容器
* docker-compose start nginx 启动nignx容器
* docker-compose restart nginx 重启项目中的nignx容器
* docker-compose run --no-deps --rm php-fpm php -v 在php-fpm中不启动关联容器，并容器执行php -v 执行完成后删除容器

Export