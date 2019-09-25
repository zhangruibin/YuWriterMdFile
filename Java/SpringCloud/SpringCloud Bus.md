# SpringCloud Bus

公众号回复：sp8，即可获取本系列的全套代码哦。

## SpringCloud bus是什么？

在上节中我们说的是SpringCloud Config，将配置文件放到GitHub之类的远端仓库，然后项目启动的时候去远端仓库拿配置。
每次修改配置文件之后需要手动重启项目才能使修改后的配置文件生效，这么做是不是很麻烦？比如我修改一个变量就要去重启项目，太浪费资源了。

而SpringCloud Bus就是来解决这个问题的，做好相关的配置之后，修改配置文件之后不需要重启项目即可做到热更，方便了许多，那是什么原理呢？

SpringCloud Bus 集成了MQ，例如rabbitmq，kafaka之类的，利用MQ的组播特性，在用户修改文件之后，给MQ一个信息：更新了配置文件信息，然后MQ会将这个信息广播至所有配置了该MQ的子项目，来进行配置文件的重新读取。

下面是SpringCloud Bus的结构图：

![](https://www.zhangruibin.com/upload/2019/09/ss2t2hk2e2jiaqeshc47gair2l.png)

## 使用方法

### 环境准备

首先我们需要准备MQ环境，具体的MQ linux安装可以参考笔者的下一篇文章，因为不想在SpringCloud系列结束的最后一节之间插入别的东西，所以讲安装mq放到了下节。

### 改造项目

#### 改造springclouddemo-11-provider-product-config-8001

* 添加SpringCloudBUs依赖：

```
<!--Bus 与 rabbitMQ依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

<!--监听器--> <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

* bootstrap.yml文件追加配置,追加后的完整配置如下：

```
spring:
 cloud: config: name: springcloud-config-product #github上的配置名称，注意没有yml后缀名
  profile: prod # 本次访问的环境配置项
  label: master # 远程库的分支名
  uri: http://localhost:5001 #Config配置中心地址，通过它获取microservice-config-product.yml配置信息    rabbitmq:
 host: 104.225.147.76
    port: 5672
    username: admin
    password: 123456
# 暴露触发消息总线的地址 management:
 endpoints: 
   web: 
     exposure: 
       include: bus-refresh
```

* 改造ProductController
类增加注解：  @RefreshScope，当增加了这个注解的时候，类才会读取更新后的配置文件。
改造后的ProductController如下：

```
package com.zhrb.springcloud.controller;

import com.zhrb.springcloud.entity.Product;
import com.zhrb.springcloud.service.ProductService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.*;

import java.util.Collection;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @ClassName ProductController
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/3 15:18
 * @Version
  */ @RestController @RefreshScope @RequestMapping("/product")
@MapperScan("com.zhrb.springcloud.mapper")
@Api(value = "/product",description = "商品管理 程序员小圈圈",position = 1)
public class ProductController {

    @Autowired
  private ProductService productService;

    @ApiOperation(value="product,返回boolean", notes="新增商品")
    @PostMapping(value = "/add")
    public boolean add(@RequestBody Product product) {
        return productService.add(product);
    }

    @ApiOperation(value="id,返回boolean", notes="新增商品")
    @GetMapping(value = "/{id}")
    public Product get(@PathVariable("id") Long id) {
        return productService.get(id);
    }

    @ApiOperation(value="product,返回boolean", notes="新增商品")
    @GetMapping(value = "/list")
    public List<Product> list() {
        List<Product> productList = productService.list();
        for (Product p :productList){
            p.setProductName(p.getProductName()+"本条数据来自8001");
        }
        return productList;
    }

/*测试修改文件常量是否立即生效*/
  @Value("${top.saler.name}")
    private String name;
    @Value("${top.saler.age}")
    private String age;
    @GetMapping(value = "/top")
    public String top() {
       return "The top saler is <b style= 'color:red'>"+age+"</b> years old,named <b style= 'color:red'>"+name+"</b>.";
    }
}
```

我们在这里面增加了一个top方法，用于获取配置文件中的值变化，返回给浏览器显示。

* 改造配置文件springcloud-config-product.yml

追加内容如下：

```
#定义全局变量，测试bus top:
saler: 
  name: Shen
    age: 60
```

经过上面的改造后，我们就可以来测试了。

## 测试
启动5001->6001->新8001项目。

* 不刷新情况下
我们在浏览器访问
http://localhost:8888/product/top

后，浏览器返回：

```
The top saler is 60 years old,named Shen.
```
此时我们将springcloud-config-product.yml的name改为Akali,age改为22.

将修改后的配置文件推送到GitHub库中，再次访问：

http://localhost:8888/product/top
浏览器返回：
```
The top saler is 60 years old,named Shen.
```


可以看到，虽然我们修改了配置文件，但是因为没有刷新，所以此时系统拿到的还是最开始的配置项。



* 刷新


  * 1.使用postman或者fiddler之类的接口测试工具发起post请求至http://localhost:8888/actuator/bus-refresh
(公众号回复：postman,即可获取postman一键安装包)

  * 2.控制台crul -X POST http://localhost:8888/actuator/bus-refresh

即可刷新接口。

刷新后我们访问：

http://localhost:8888/product/top

此时浏览器返回：

```
The top saler is 22 years old,named Akali.
```

读取的是我们新设置的参数，热更成功。

后记：

在安装过rabbitmq之后，我们启动项目的时候有可能会启动报错，大致是连接rabbitMQ被拒绝：Caused by: com.rabbitmq.client.AuthenticationFailureException: ACCESS_REFUSE，这时候我们要登录时上rabbitmq的控制台去查看配置的账号权限是不是denied，默认是拒绝远程链接的，需要我们手动改一下，具体的网上有很多分享的，这里就不细说了。

截止到这节，一代的SpringCloud入门我们基本已经完事了，说的比较浅显，很多东西需要自己去实战应用才能发现不足去弥补。

感谢大家的观看。