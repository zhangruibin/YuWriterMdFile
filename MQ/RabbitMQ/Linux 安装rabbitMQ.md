# Linux 安装rabbitMQ


安装rabbitMQ需要erlang环境，所以要先在主机上安装erlang并设置环境变量。

## 安装erlang

### 下载安装包

#### 官网

[http://erlang.org/download/](http://erlang.org/download/)
在官网下载想要的版本的安装包，大概230M，下载速度较慢，有百度会员的朋友推荐公众号方式。

#### 公众号

公众号回复：erlang.
即可或许2.0.3版的。

### 安装

gz包上传到服务器tmp目录下，进入到tmp目录进行安装。
```
# cd /tmp
# mkdir -p /usr/local/erlang
# tar -xvf otp_src_20.3.tar.gz
# cd otp_src_20.1
# ./configure --prefix=/usr/local/erlang --with-ssl --enable-threads --enable-smp-support --enable-kernel-poll --enable-hipe --without-javac
# make -j8
# make install
```

注意，官网的安装包没有用gzip格式压缩，所以不用加z指令，在解压的时候参数用xvf就行，否则报错：
gzip：stdin：not in gzip format错误。

### 设置环境变量

```
# vim /etc/profile
```

在末尾加入以下内容：
```
#set erlang environment
export PATH=$PATH:/usr/local/erlang/bin
```

wq保存退出之后，使环境变量生效：
```
# source /etc/profile
```

###  验证

```
# erl
```

出现如下输出证明安装及设置环境变量成功：
```
Erlang/OTP 20 [erts-9.3] [source] [64-bit] [smp:2:2] [ds:2:2:10] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V9.3  (abort with ^G)
1>
```


## 安装rabbitMQ （单机版）

### 下载安装包

#### 官网
下载速度还行，文件比较小
[http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.15/rabbitmq-server-generic-unix-3.6.15.tar.xz](http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.15/rabbitmq-server-generic-unix-3.6.15.tar.xz)

#### 公众号

公众号回复：rabbitmq,即可获取下载资源。

### 安装
老惯例先将文件上传至服务器tmp目录，然后解压并且mv：
```
# tar -xvf rabbitmq-server-generic-unix-3.6.15.tar.xz
# mv rabbitmq_server-3.6.15 /usr/local/RabbitMQ
```

### 设置环境变量

```
# vim /etc/profile
```
在末尾加入以下内容：
```
#set RabbitMQ environment
export PATH=$PATH:/usr/local/RabbitMQ/sbin
```


wq保存后使环境变量生效：

```
source /etc/profile
```


### 启动rabbitMQ
#### 启用WEB管理插件

```
# cd /usr/local/RabbitMQ/sbin
查看插件列表
# ./rabbitmq-plugins list
# ./rabbitmq-plugins enable rabbitmq_management
```

#### 后台运行

```
# ./rabbitmq-server -detached
```

### 开放端口

centos7 iptables开放端口方法如下：
#### 编辑文件
```
vim /ets/sysconfig/iptanles
```

增加端口：
```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 5372 -j ACCEPT

```


保存规则生效：
```
 systemctl restart iptables

```

### 远程访问

值得一提的是，按照上面的那套操作下来，我们是无法在远程登录guest用户的，所以我们新建一个用户用以登录：

```
添加用户：
# ./rabbitmqctl add_user username password 如：./rabbitmqctl add_user admin 123456789
授权用户管理员： # ./rabbitmqctl set_user_tags admin administrator
如：./rabbitmqctl set_user_tags admin administrator
添加虚拟机： # ./rabbitmqctl add_vhost vhostname 如：./rabbitmqctl add_vhost admin_vhost
授权用户到虚拟机： # ./rabbitmqctl set_permissions -p vhostname username ".*" ".*" ".*"
如：./rabbitmqctl set_permissions -p admin_vhost admin ".*" ".*" ".*"
```
然后浏览器访问：

ip:15672

然后输入账号密码即可登录。