# SpringCloud Feign

**公众号回复sp5,即可获取截至到本节的代码哦~~**​

Feign是Netflix公司开源的轻量级Rest客户端( https://github.com/OpenFeign/feign )，使用 Feign 可以非常方
便、简单的实现 Http 客户端，使用 Feign 只需要定义一个接口，然后在接口上添加注解即可。
Spring Cloud 对 Feign 进行了封装，Feign 默认集成了 Ribbon 实现了客户端负载均衡调用。
目前大家更习惯用面向接口编程，比如 Service接口，Dao接口等，这是大家默认遵守的规范
微服务间的调用就有两种方式：
1. 通过微服务名称，获得服务的调用地址
2. 通过接口+注解，获得服务的调用 ——Feign （为适应业界其它程序员提出的，还是遵循面向接口编程）
类似于以前Mapper接口上使用@Mapper注解进行标识，而使用Feign就只要在接口上标注@FeignClient注解。

总的来说：
*   Feign 采用的是基于接口的注解
*   Feign 整合了ribbon，具有负载均衡的能力
*   整合了Hystrix，具有熔断的能力




## 复制数据库为springcloud_02

将数据库  springloud_db01 复制一个数据库命名为 springcloud_db02
,并且将数据库来源字段改为springcloud_db02。并且修改8012的db配置文件为springcloud_db02。
## 复制消费者8002项目

复制8002项目，项目号及端口号改为8012，然后修改8012pom,并将其引入到主项目pom，启动验证无误。



## 修改8012项目
### 整合Ribbon实现接口访问及负载均衡
### 修改pom文件，增加
增加依赖：
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

```

#### 1.修改application.yml
为了后续的feign使用，增加feign配置：

```
server:
 port: 8012

eureka:
 client: serviceUrl: defaultZone: http://127.0.0.1:9000/eureka/,http://127.0.0.1:9002/eureka/
feign:
 hystrix: enabled: true #此处关于hystrix的配置一定要配置上，否则Feign的FallBack设置无效
  client:
 config: default: connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
hystrix:
 command: default: execution: isolation: strategy: SEMAPHORE
spring:
 application: name: consumer-product-feign
```

并且修改springname 为consumer-product-feign。

#### 2.修改ProductConsumerController.java
```
package com.zhrb.springcloud.controller;

import com.zhrb.springcloud.entity.Product;
import com.zhrb.springcloud.service.ConsumerService;
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
  */ 
  @RestController 
  @RequestMapping(value = "/consumer/product")
public class ProductConsumerController {

    @Autowired(required = false)
    private ConsumerService consumerService;

    @PostMapping(value = "/add")
    public boolean add(Product product) {
        return consumerService.add(product);
    }

    @GetMapping(value = "/get/{id}")
    public Product get(@PathVariable("id") Long id) {
        return consumerService.get(id);
    }

    @GetMapping(value = "/list")
    public List<Product> list() {
        return consumerService.list();
    }
}
```


#### 修改ConsumerService.java
```
package com.zhrb.springcloud.service;

import com.zhrb.springcloud.entity.Product;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import java.util.List;

/**
 * @ClassName ConsumerService
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/6 16:49
 * @Version
  */ 
 @FeignClient(value = "springcloud-product",fallback = FeignFallBack.class)
//@Service 
public interface ConsumerService {
    @RequestMapping(value = "/product/get/{id}",method = RequestMethod.GET)
    Product get(Long id);
    @RequestMapping(value = "/product/list",method = RequestMethod.GET)
    List<Product> list();
    @RequestMapping(value = "/product/add",method = RequestMethod.POST)
    boolean add(Product product);
}
```

#### 新建FeignFallBack .java
```
package com.zhrb.springcloud.service;

import com.zhrb.springcloud.entity.Product;
import org.springframework.stereotype.Component;

import java.util.*;

/**
 * @ClassName FeignFallBack
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/9 10:42
 * @Version
*/ 
@Component public class FeignFallBack implements ConsumerService{
Product defultFallback = new Product();
    @Override
  public Product get(Long id) {
        return defultFallback;
    }

    @Override
  public List<Product> list() {
       List list = new ArrayList(1);
        list.add(defultFallback);
        System.out.println("服务中断，降级到本方法，控制台打印bilibilibiibilibilibilib ili");
        return list;
    }

    @Override
  public boolean add(Product product) {
        return false;
    }

}
```

#### 修改启动类SpringCloudProductConsumer_8012.java
```
package com.zhrb.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
import org.springframework.cloud.openfeign.EnableFeignClients;

/**
 * @ClassName SpringCloudProductConsumer_8012
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/4 11:59
 * @Version
  */   
@SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
@EnableEurekaServer 
@EnableDiscoveryClient 
@EnableFeignClients public class SpringCloudProductConsumer_8012_Feign {
public static void main(String[] args) {
        SpringApplication.run(SpringCloudProductConsumer_8012_Feign.class, args);
    }
}
```
#### 启动项目验证
启动顺序：
（注册中心）9000->9002->（服务提供者）8001->8011->（消费者）8012
启动后访问注册中心及8012效果如下，可以看出也能达到Ribbon那样的负载均衡。
![](https://www.zhangruibin.com/upload/2019/09/5tjs77gr76hrpqjo09c53c9j80.png)
![](https://www.zhangruibin.com/upload/2019/09/0dhkheti64gneohs8elca36aqb.png)
![](https://www.zhangruibin.com/upload/2019/09/s7p4nc208gis3osot0hdmrbe1i.png)

此时如果将其中一个服务提供者，比如8011挂了，再次多次访问效果如下：

![](https://www.zhangruibin.com/upload/2019/09/pcj82q5j3egplqb5ck2bd12n7g.png)
至此，feign的基本使用已经展示完毕，更详细的内容以及替代品，博主会在接下来的文章中进行更新，请各位看官小板凳准备好~~~~


