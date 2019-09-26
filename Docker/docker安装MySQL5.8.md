<blockquote>
    欢迎关注微信公众号：<a href="" rel="nofollow" target="_blank">程序员小圈圈</a><br>
  转载请标明出处^_^<br> 原文首发于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">www.zhangruibin.com</a><br> 本文出自于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">RebornChang的博客</a>
</blockquote>
 
 # docker安装MySQL5.8
 ## 下载镜像
 ```
 docker pull mysql
 ```
 ## 启动
 ```
  docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=YourPWD123! -d mysql
 ```
 注意：如果本机安装了一个MySQL实例，那么此时的需要先将下载下来的MySQL镜像改个名字再启动，比如博主的再主机上安装了MySQL5.6的，所以在直接使用上面的命令启动的时候会报错。应该如下操作。
 ### 修改镜像名字为mysql2 
 ```
 docker tag mysql:latest mysql2:v2
 ```
 ### 启动修改后的镜像
 ```
 docker run --name mysql2 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=YourPWD12! -d mysql
 注意端口不要跟本地的冲突
 ```
 ## 镜像中登录MySQL
 ```
 mysql -u root -p
 
 执行命令之后输入对应的密码即可登录到镜像中的MySQL
 ```
 ![](https://www.zhangruibin.com/upload/2019/06/45mm4hv1jsj2grc7ve5tvq4qou.png)
 ## 查看MySQL版本
 ```
 select version();
 可以看到输出：
 +-----------+
| version() |
+-----------+
| 8.0.16    |
+-----------+
1 row in set (0.02 sec)
 ```
 ## 修改
 ALTER USER 'root'@'localhost' IDENTIFIED BY 'YourPWD12!';

 ## 添加远程登录用户
CREATE USER 'rebornchang'@'%' IDENTIFIED WITH mysql_native_password BY 'YourPWD12!';
GRANT ALL PRIVILEGES ON *.* TO 'rebornchang'@'%';

## 开启对应端口(iptables)
```
1.编辑文件
vim /etc/sysconfig/iptables
2.增加内容
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3307 -j ACCEPT
3.重启生效
service iptables restart

```
## 测试连接
经测试，连接成功：
![](https://www.zhangruibin.com/upload/2019/06/t84rv7ra9gjrtq98j57vfpk699.png)
 
 值得一提的是，当这样启动了镜像里面的MySQL5.8实例之后，本地博主安装的MySQL5.6不能用了，回来研究下是哪里的问题。

    
 **亲**，博主的微信公众号   
 ‘**程序员小圈圈**’开始持续更新了哟~~   
 识别二维码或者直接搜索名字  ‘**程序员小圈圈**’   即可关注本公众号哟~~   
 **不只是有技术哟~~** 
  **还有很多的学习娱乐资源免费下载哦~~** 
  还可以学下教育知识以及消遣娱乐哟~~
  **求关注哟~~**     ![](https://www.zhangruibin.com/upload/2019/08/6re5uggg4khmtrrit534ubh759.png)'