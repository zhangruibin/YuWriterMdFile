# SpringCloud Ribbon

**公众号发送：sp4，即可获取截至本节的代码哦~**
## Ribbon是什么

Spring Cloud Ribbon 是基于 Netflix 公司发布的开源项目 Ribbon 进行封装的一套客户端负载均衡器 。

github地址：https://github.com/Netflix/ribbon

Ribbon 从 Eureka Server 获取服务列表，Ribbon根据负载均衡算法直接请求到具体的微服务，中间省去了负载均
衡服务。
下图是Ribbon负载均衡的流程图简图：
![](https://www.zhangruibin.com/upload/2019/09/5ua4qaca26imipnf9nd1t5dj2j.png)

在消费者微服务中使用 Ribbon 实现负载均衡，Ribbon 先从EurekaServer中获取服务列表,然后再根据负载均衡的算法（默认轮训算法）去调用微服务。

## Ribbon怎么用

由于依赖了spring-cloud-starter-netflix-eureka-client，会自动添加spring-cloud-starter-netflix-ribbon依赖。


然后我们就可以来在旧的项目上进行改造了。
## 改造项目
先回顾一下我们的基础项目：
1.项目01，父级项目；
2.项目02，子级公共项目；
3.项目03，子级项目product 8001；
4.项目04，子级项目consumer8002；
5.项目05，子级项目eureka server9000；
6.项目06，子级项目eureka server9002.

**首先：**
**改造服务消费者8002项目**
1.修改ConfigBean.java
在ConfigBean.java 中加入注解@LoadBalanced：
```
package com.zhrb.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * @ClassName ConfigBean
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/4 11:56
 * @Version
  */ 
  @Configuration
   public class ConfigBean {
    // 向容器中添加 RestTemplate 组件,直接通过此组件可调用 REST 接口
 ///@LoadBalanced表示这个RestTemplate开启负载均衡，在调用服务提供者的接口时， //可使用 服务名称 替代真实IP地址。服务名称 就是服务提供者在application.yml中 //配置的spring.application.name属性的值  @LoadBalanced
 @Bean  public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```


2.修改ProductConsumerController
PS：之前手误写成了ProductControllerConsumer，现在改回来
将静态变量： REST_URL_PREFIX 的值修改为：http://springcloud-product

修改后的文件内容如下：
```
package com.zhrb.springcloud.controller;

import com.zhrb.springcloud.entity.Product;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.client.RestTemplate;

import java.util.List;

/**
 * @ClassName ProductControllerConsumer
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/4 11:57
 * @Version
  */ @RestController @RequestMapping(value = "/consumer/product")
public class ProductConsumerController {

    //private static final String REST_URL_PREFIX = "http://127.0.0.1:8001";
 //修改为注册中心的地址springcloud-product //private static final String REST_URL_PREFIX = "http://127.0.0.1:9000";  private static final String REST_URL_PREFIX = "http://springcloud-product";
    @Autowired
  private RestTemplate restTemplate;

    @PostMapping(value = "/add")
    public boolean add(Product product) {
        return restTemplate.postForObject(REST_URL_PREFIX + "/product/add",
                product, Boolean.class);
    }

    @GetMapping(value = "/{id}")
    public Product get(@PathVariable("id") Long id) {
        return restTemplate.getForObject(REST_URL_PREFIX + "/product/" + id,
                Product.class);
    }

    @GetMapping(value = "/list")
    public List<Product> list() {
        return restTemplate.getForObject(REST_URL_PREFIX + "/product/list",
                List.class);
    }
}
```


@LoadBalanced表示这个RestTemplate开启负载均衡，在调用服务提供者的接口时，可使用 服务名称替代真实IP地址。服务名称 就是服务提供者在application.yml中配置的spring.application.name属性的值 。

**然后：**
修改8001项目中的ProductController.java
修改list()方法，目的是能在前台看出来消费者消费的是哪个服务提供者，修改后ProductController代码如下：
```
package com.zhrb.springcloud.controller;

import com.zhrb.springcloud.entity.Product;
import com.zhrb.springcloud.service.ProductService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.Collection;
import java.util.List;

/**
 * @ClassName ProductController
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/3 15:18
 * @Version
  */ @RestController @RequestMapping("/product")
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
}
```




**最后：**

在这些的基础上，我们直接复制项目04，增加一个服务提供者。

像之前复制9000项目那样，复制项目并修改文件。
新项目命名为springclouddemo-03-provider-product-8011，修改apllication.yml文件的端口声明为8011，不修改spring name（重点）。

复制完项目之后，将ProductController中的list（）方法的‘来自8001’改为‘来自8011’。

此时系统架构简图应该如下图所示：

![](https://www.zhangruibin.com/upload/2019/09/uof1hu4teci8go6vkeqp4da6sc.png)
这样理论上来讲，消费者从开启了LoadBalanced ribbon 去均衡的访问8001，以及8011项目。



那么事实是不是这样呢？我们试下就知道了：
### 启动项目验证

项目启动顺序为： 
9000->9002->8001->8011->8002.

启动后访问其中一个Eureka Server的地址：http://localhost:9002
可以看到，两个服务提供者和一个服务消费者都已经注册了进来：
![](https://www.zhangruibin.com/upload/2019/09/v86qki6nr2ga1plbmncm0prf08.png)
然后我们访问服务消费者去消费服务：
http://localhost:8002/consumer/product/list
第一次访问：
![](https://www.zhangruibin.com/upload/2019/09/sqb6ci0k1cggjovtvcu6pq8l6c.png)
第二次访问：
![](https://www.zhangruibin.com/upload/2019/09/02vrj6fvqggeipdb2u876b5r86.png)
从上面两张图我们可以看出来，两次访问的是不同的服务提供者，LoadBalance成功。

最后我们总结一下，用户点击下按钮后后台的操作是怎样一个步骤：
0.几个项目启动，8001,8011,8002分别向9000,9002项目注册；
1.用户发起请求，请求路由到消费者8002,；
2.8002发起请求去请访问项目名为springcloud-product的服务提供者；
3.ribbon根据负载均衡，获取可以实现访问的可提供访问的服务列表，然后根据负载均衡算法，将8002发起的消费请求转发到8011或者8001项目进行访问；
4.服务方返回数据给8002,8002展示给用户。
至此，关于ribbon的基本应用及效果已经展示完毕，值得一提的是，还有一个组件和Ribbon效果类似，那就是Feign。
下节我将开一个小节，将Ribbon换成Feign，并对两者进行一个简单的对比。

**感兴趣的小伙伴可以关注下公众号的推送或者博主的技术博客哦。**






