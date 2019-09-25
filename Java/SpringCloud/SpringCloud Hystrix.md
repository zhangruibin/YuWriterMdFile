# SpringCloud Hystrix


在微服务架构中，根据业务来拆分成一个个的服务，而服务与服务之间存在着依赖关系 （比如用户调商品，商品调库存，库存调订单等等），在Spring Cloud中多个微服务之间可以用 RestTemplate+Ribbon 和 Feign来调用。
在服务之间调用的链路上由于网络原因、资源繁忙或者自身的原因，服务并不能保证100%可用，如果单个服务出现问题，调用这个服务就会出现线程阻塞，导致响应时间过长或不可用，此时若有大量的请求涌入，容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重
后果，这就是服务故障的“雪崩”效应。

因此我们这节中说这么一个东西：Hystrix。

先说下什么是熔断：
熔断机制是应对雪崩效应的一种微服务链路保护机制。在微服务架构中，一个请求需要调用多个服务是非常常见的。
当服务之间调用的链路上某个微服务不可用或者响应时间太长时，会导致连锁故障。当失败的调用到一定阈值（缺省是5秒内20次调用失败) 就会启动熔断机制。在 SpringCloud 框架里熔断机制通过Hystrix实现，Hystrix会监控微服务间调用的状况。

## Hystrix特性

　　1.请求熔断： 当Hystrix Command请求后端服务失败数量超过一定比例(默认50%), 断路器会切换到开路状态(Open). 这时所有请求会直接失败而不会发送到后端服务. 断路器保持在开路状态一段时间后(默认5秒), 自动切换到半开路状态(HALF-OPEN).

　　　 这时会判断下一次请求的返回情况, 如果请求成功, 断路器切回闭路状态(CLOSED), 否则重新切换到开路状态(OPEN). Hystrix的断路器就像我们家庭电路中的保险丝, 一旦后端服务不可用, 断路器会直接切断请求链, 避免发送大量无效请求影响系统吞吐量, 并且断路器有自我检测并恢复的能力.

　　2.服务降级：Fallback相当于是降级操作. 对于查询操作, 我们可以实现一个fallback方法, 当请求后端服务出现异常的时候, 可以使用fallback方法返回的值. fallback方法的返回值一般是设置的默认值或者来自缓存.告知后面的请求服务不可用了，不要再来了。

　　3.依赖隔离(采用舱壁模式，Docker就是舱壁模式的一种)：在Hystrix中, 主要通过线程池来实现资源隔离. 通常在使用的时候我们会根据调用的远程服务划分出多个线程池.比如说，一个服务调用两外两个服务，你如果调用两个服务都用一个线程池，那么如果一个服务卡在哪里，资源没被释放

　　　后面的请求又来了，导致后面的请求都卡在哪里等待，导致你依赖的A服务把你卡在哪里，耗尽了资源，也导致了你另外一个B服务也不可用了。这时如果依赖隔离，某一个服务调用A B两个服务，如果这时我有100个线程可用，我给A服务分配50个，给B服务分配50个，这样就算A服务挂了，我的B服务依然可以用。

　　4.请求缓存：比如一个请求过来请求我userId=1的数据，你后面的请求也过来请求同样的数据，这时我不会继续走原来的那条请求链路了，而是把第一次请求缓存过了，把第一次的请求结果返回给后面的请求。

　　5.请求合并：我依赖于某一个服务，我要调用N次，比如说查数据库的时候，我发了N条请求发了N条SQL然后拿到一堆结果，这时候我们可以把多个请求合并成一个请求，发送一个查询多条数据的SQL的请求，这样我们只需查询一次数据库，提升了效率。

## Hystrix流程

1:每次调用创建一个新的HystrixCommand,把依赖调用封装在run()方法中.
2:执行execute()/queue做同步或异步调用.
4:判断熔断器(circuit-breaker)是否打开,如果打开跳到步骤8,进行降级策略,如果关闭进入步骤5.
5:判断线程池/队列/信号量是否跑满，如果跑满进入降级步骤8,否则继续后续步骤6.
6:调用HystrixCommand的run方法.运行依赖逻辑
6a:依赖逻辑调用超时,进入步骤8.
7.0:判断逻辑是否调用成功
7.1:返回成功调用结果
7.2:调用出错，进入步骤8.
8:计算熔断器状态,所有的运行状态(成功, 失败, 拒绝,超时)上报给熔断器，用于统计从而判断熔断器状态.
9.0:getFallback()降级逻辑.以下四种情况将触发getFallback调用：
　　　　(9.0.1):run()方法抛出非HystrixBadRequestException异常。
　　　　(9.0.2):run()方法调用超时
　　　　(9.0.3):熔断器开启拦截调用
　　　　(9.0.4):线程池/队列/信号量是否跑满
9.2:没有实现getFallback的Command将直接抛出异常
9.3:fallback降级逻辑调用成功直接返回
9.4:降级逻辑调用失败抛出异常
10:返回执行成功结果

上节中我们用过了Feign及Hystrix实现了客户端的熔断来进行服务降级，这节我们了解一下服务端的熔断机制。

## 改造8011项目

因为我们之前有两个服务端项目8001以及8011了，所以这次我们就不新建项目了，在原有的8011项目上进行修改，当然，有兴趣的小伙伴可以复制8011项目修改。

1.pom.xml增加
```
<!-- 添加对hystrix断路器的依赖--> <dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-core</artifactId>
    <version>1.5.6</version>
</dependency>
```

2.启动类增加注解@EnableHystrix

3.修改ProductController.java
```
package com.zhrb.springcloud.controller;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.zhrb.springcloud.entity.Product;
import com.zhrb.springcloud.service.ProductService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * @ClassName ProductController
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/3 15:18
 * @Version
  */ 
 @RestController @RequestMapping("/product")
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
    //当get方法出现异常时，则会调用hystrixGet方法处理
  @HystrixCommand(fallbackMethod = "fallbackMethod")
    public Product get(@PathVariable("id") Long id) {
        //因为数据库中没有id为负值的商品，所以当有负值id传进来时，我们当做是恶意请求，或者是需要处理的特定流程来模拟
  if (id < 0){
            throw new RuntimeException("ID=" + id + "无效");
        }else {
            return productService.get(id);
        }
    }

    @ApiOperation(value="product,返回boolean", notes="新增商品")
    @GetMapping(value = "/list")
    public List<Product> list() {
        List<Product> productList = productService.list();
        for (Product p :productList){
            p.setProductName(p.getProductName()+"本条数据来自8011");
        }
        return productList;
    }
    public Product fallbackMethod( Long id) {
        Product p = new Product();
        p.setPid(id);
        p.setProductName("无效，进行服务降级处理");
        p.setDbSource("!");
        return p;
    }
}
```

4.修改application.xml eureka 的instanceId 值为：
```${spring.application.name}:${server.port}-hystrix 
```

### 启动验证
启动顺序：9000->9002->8001->8011->8002->8012。

因为8012项目我们之前做了客户端Feign熔断，所以我们访问8002项目，效果如下图所示：
![](https://www.zhangruibin.com/upload/2019/09/7edb6qjp72ikqqm6jsbi8gu7ao.png)
![](https://www.zhangruibin.com/upload/2019/09/70sme3psu6ha8o717sno1t8nsv.png)
![](https://www.zhangruibin.com/upload/2019/09/rtkeqc0mrsj59pobqet2kjnqei.png)

可以看到，当访问
http://localhost:8002/consumer/product/-1
时，已经做了降级处理，返回给前台我们所定义的值。


还有很多东西值得深入探讨，本节就不再多说，之后会挨个组件深入研究下。
下节说hystrix-dashboard，敬请期待，关注公众号可以第一时间接收到推送哦~~


