# docker常用命令整理
<blockquote>
    欢迎关注微信公众号：<a href="" rel="nofollow" target="_blank">程序员小圈圈</a><br>
  转载请标明出处^_^<br> 原文首发于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">www.zhangruibin.com</a><br> 本文出自于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">RebornChang的博客</a>
</blockquote>

总的来说很多命令跟linux的还是很接近的，只是前面要加上docker标志，看下文体会下吧。
## docker 的一些常用命令
### 显示可用的容器
```
docker images 
```
### 删除指定镜像
```
docker rmi <镜像Id> 
```
### 下载镜像
```
docker pull hello-world 
```
不指定版本号默认拉取latest版本的
### 删除指定镜像
```
docker rmi <镜像Id> 
```
### 查看容器
```
docker ps   [OPTIONS]
```
列出当前正在运行的容器, 结果的第一列是container_Id, 第2列是容器名称. 
参数：
*   -a :显示所有的容器，包括未运行的。

*   -f :根据条件过滤显示的内容。

*   --format :指定返回值的模板文件。

*  -l :显示最近创建的容器。

*   -n :列出最近创建的n个容器。

*  --no-trunc :不截断输出。

*  -q :静默模式，只显示容器编号。

*   -s :显示总的文件大小。
### 停止指定的容器
```
docker stop container_id/container-name 
 该容器Id或名称可以从docker ps中获取. 
```
### 启动容器
```
docker start container_id/container-name
 该容器Id或名称可以从docker ps中获取. 
```
### 重启容器
```
docker restart container_id/container-name 
 该容器Id或名称可以从docker ps中获取. 
```

### 删除容器

```
docker rm container_id/container-name
```
#### 批量删除容器
```
docker rm $(docker ps -a -q)
 删除所有运行结束了容器, 正在运行的容器不会被删除
 
```
### 查看容器内的进程
```
docker top container_id/container-name
```

###  查看容器的日志输出
```
docker logs [-f] [-t] [--tail string] 容器名, 查看容器的日志输出, -f是打开跟踪, -t是加上时间戳, --tail 100 表示仅显示最后的100行日志
```
*   -f : 跟踪日志输出

*   --since :显示某个开始时间的所有日志

*   -t : 显示时间戳

*   --tail :仅列出最新N条容器日志
### 搜寻镜像
```
docker search 镜像名字
```
### 显示指定镜像的详细信息
```
docker image inspect image_id 

docker container inspect container_id/container-name 
 （包括容器的Ip）
```
### 列出没有被容器化的镜像

```
docker images -f dangling=true
```
###  删除那些没有被容器化的镜像
```
docker rmi $(docker images -qf dangling=true) 
```
### 可以磁盘占用情况.
```
docker system df 
``` 



## docker 一些管理命令集

除了上面常用的命令外, docker 还有一些管理命令集, 这些命令集还可以包含二级命令:
```
config Manage Docker configs
container Manage containers
image Manage images
network Manage networks
node Manage Swarm nodes
plugin Manage plugins
secret Manage Docker secrets
service Manage services
stack Manage Docker stacks
swarm Manage Swarm
system Manage Docker
trust Manage trust on Docker images
volume Manage volumes
```

比较常用的是:
```
docker image build  编译 Dockfile
docker network create 创建 docker 网络
docker volume create 创建数据卷
```

## docker run/exec 命令

运行 hello-world 容器, 如果本地没有下载, 将会自动从hub站点下载. 
```
docker run hello-world 命令
```
以守护态运行容器
```
docker run -d --name mybusybox busybox /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

登陆一个容器, 运行中的容器其实是**一个功能完备的Linux操作系统**, 所以我们可以在登陆该容器执行常规的Linux命令. 
```
docker exec -it container_id/container-name /bin/bash
```



使用 redis-cli 登陆 myredis 容器
```
docker exec -it myredis redis-cli 
```



exec 后的 -it 参数的意思是, 以交互的方式并分配一个伪tty, 经常一起联用.

附录：

菜鸟教程
# Docker 命令大全

### 容器生命周期管理

*   [run](https://www.runoob.com/docker/docker-run-command.html)
*   [start/stop/restart](https://www.runoob.com/docker/docker-start-stop-restart-command.html)
*   [kill](https://www.runoob.com/docker/docker-kill-command.html)
*   [rm](https://www.runoob.com/docker/docker-rm-command.html)
*   [pause/unpause](https://www.runoob.com/docker/docker-pause-unpause-command.html)
*   [create](https://www.runoob.com/docker/docker-create-command.html)
*   [exec](https://www.runoob.com/docker/docker-exec-command.html)

### 容器操作

*   [ps](https://www.runoob.com/docker/docker-ps-command.html)
*   [inspect](https://www.runoob.com/docker/docker-inspect-command.html)
*   [top](https://www.runoob.com/docker/docker-top-command.html)
*   [attach](https://www.runoob.com/docker/docker-attach-command.html)
*   [events](https://www.runoob.com/docker/docker-events-command.html)
*   [logs](https://www.runoob.com/docker/docker-logs-command.html)
*   [wait](https://www.runoob.com/docker/docker-wait-command.html)
*   [export](https://www.runoob.com/docker/docker-export-command.html)
*   [port](https://www.runoob.com/docker/docker-port-command.html)

### 容器rootfs命令

*   [commit](https://www.runoob.com/docker/docker-commit-command.html)
*   [cp](https://www.runoob.com/docker/docker-cp-command.html)
*   [diff](https://www.runoob.com/docker/docker-diff-command.html)

### 镜像仓库

*   [login](https://www.runoob.com/docker/docker-login-command.html)
*   [pull](https://www.runoob.com/docker/docker-pull-command.html)
*   [push](https://www.runoob.com/docker/docker-push-command.html)
*   [search](https://www.runoob.com/docker/docker-search-command.html)

### 本地镜像管理

*   [images](https://www.runoob.com/docker/docker-images-command.html)
*   [rmi](https://www.runoob.com/docker/docker-rmi-command.html)
*   [tag](https://www.runoob.com/docker/docker-tag-command.html)
*   [build](https://www.runoob.com/docker/docker-build-command.html)
*   [history](https://www.runoob.com/docker/docker-history-command.html)
*   [save](https://www.runoob.com/docker/docker-save-command.html)
*   [load](https://www.runoob.com/docker/docker-load-command.html)
*   [import](https://www.runoob.com/docker/docker-import-command.html)

### info|version

*   [info](https://www.runoob.com/docker/docker-info-command.html)
*   [version](https://www.runoob.com/docker/docker-version-command.html)


 **亲**，博主的微信公众号   
 ‘**程序员小圈圈**’开始持续更新了哟~~   
 识别二维码或者直接搜索名字  ‘**程序员小圈圈**’   即可关注本公众号哟~~   
 **不只是有技术哟~~** 
  **还有很多的学习娱乐资源免费下载哦~~** 
  还可以学下教育知识以及消遣娱乐哟~~
  **求关注哟~~**     ![](https://www.zhangruibin.com/upload/2019/08/6re5uggg4khmtrrit534ubh759.png)'

