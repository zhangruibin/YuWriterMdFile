# SpringCloud Eureka集群
<blockquote>
欢迎关注微信公众号：<a href="" rel="nofollow" target="_blank">程序员小圈圈</a><br>原文首发于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">www.zhangruibin.com</a><br> 本文出自于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">RebornChang的博客</a><br>
转载请标明出处^_^<br>
</blockquote>

公众号回复：sp3,即可获取本节代码哦~

## 为什么要集群

为了避免 Eureka Server的失效，Eureka Server 高可用环境需要部署两个及以上Eureka Server，它们互相向对方注册。
在实际使用时Eureka Server至少部署两台服务器，实现高可用，集群的原理是两台Eureka Server互相注册然后微服务需要连接两台Eureka Server注册，当其中一台Eureka死掉也不会影响服务的注册与发现。
## 搭建集群

### 创建model9002
直接复制9000项目，然后把项目中能看到的9000全部换成9002，包括配置文件以及文件命名，然后，修改主项目pom文件中的</modules > 添加9002的< module >即可。

完成上面的操作之后，启动并访问无误，那么接下来再修改配置文件。
此时项目截图如下：

![](https://www.zhangruibin.com/upload/2019/09/4vpq1sp6kmjd3remctb0hstrs0.png)
### 修改配置文件

经过上面那些步骤，现在需要修改的配置文件就只有一个了，那就是application.yml

依次修改8001，8002，9000，9002的application.yml文件中的
eureka项的defaultZone字段值为：
```
http://127.0.0.1:9000/eureka/,http://127.0.0.1:9002/eureka/

http://127.0.0.1:9000/eureka/,http://127.0.0.1:9002/eureka/

http://127.0.0.1:9002/eureka/


http://127.0.0.1:9000/eureka/
```

此时的项目Eureka架构图如下图所示：

![](https://www.zhangruibin.com/upload/2019/09/v4pp9f714ojq0ohohfum1outsb.png)
## 启动项目验证

配置完成之后，依次启动项目：9000->9002->8001->8002

启动完成之后，浏览器输入：
http://localhost:9000
http://localhost:9002

可以看到效果如下图所示：
![](https://www.zhangruibin.com/upload/2019/09/23opprckh4i6foq2ipch08h3g6.png)

![](https://www.zhangruibin.com/upload/2019/09/qstmkenqici68p9krh4gcevuia.png)
此时断开9000或者9002项目，另一个依然可用，其实也就是做了一个负载容灾，还有更多的配置可以挖掘，有兴趣的小伙伴可以自行挖掘。本节就到这里结束了，比较短，下节说Ribbon感兴趣的小伙伴可以关注下公众号的推送或者博主的技术博客哦。

 **亲**，博主的微信公众号   
 ‘**程序员小圈圈**’开始持续更新了哟~~   
 识别二维码或者直接搜索名字  ‘**程序员小圈圈**’   即可关注本公众号哟~~   
 **不只是有技术哟~~** 
  **还有很多的学习娱乐资源免费下载哦~~** 
  还可以学下教育知识以及消遣娱乐哟~~
  **求关注哟~~**     ![](https://www.zhangruibin.com/upload/2019/08/6re5uggg4khmtrrit534ubh759.png)'


