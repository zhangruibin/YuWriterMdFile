# SpringCloud Zuul


**公众号回复：sp6，即可获取截至本节的代码哦~**

**Spring Cloud Zuul** 是整合Netflix公司的 Zuul开源项目（官方：https://github.com/Netflix/zuul）；
Zuul 包含了对请求路由和校验过滤两个最主要的功能：
其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础，
客户端请求网关/api/product，通过路由转发到 product 服务
客户端请求网关/api/order，通过路由转发到 order 服务
而过滤功能则负责对请求的处理过程进行干预，是实现请求校验等功能的基础.
Zuul 和 Eureka 进行整合，将 Zuul 自身注册为 Eureka 服务治理中的服务，同时从 Eureka 中获得其他微服
务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。
**注意：Zuul服务最终还是会注册进Eureka**


## 新建项目 springclouddemo-07-zuul-gateway-7777
pom.xml
```
<?xml version="1.0" encoding="UTF-8"?> <project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud-demo-01</artifactId>
        <groupId>com.zrb.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>springclouddemo-07-zuul-gateway-7777</artifactId>
    <dependencies>
        <!--zuul路由网关依赖-->
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
            <version>1.3.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.SR2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>
```

启动类：ZuulGateway_7777.java
```
package com.zhrb.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

/**
 * @ClassName ZuulGateway_6000
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/10 17:21
 * @Version
  */

    @EnableEurekaClient
     @SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
public class ZuulGateway_7777 {
    public static void main(String[] args) {
        SpringApplication.run(ZuulGateway_7777.class,args);
    }
}
```

application.yml
```
server:
 port: 7777 # 服务端口 spring:
 application: name: zuul-gateway
eureka:
 client: registerWithEureka: true # 服务注册开关
  fetchRegistry: true # 服务发现开关
  serviceUrl: # 客户端(服务提供者)注册到哪一个Eureka Server服务注册中心，多个用逗号分隔
  defaultZone: http://127.0.0.1:9000/eureka/,http://127.0.0.1:9002/eureka/
    instance:
 instanceId: ${spring.application.name}:${server.port} # 指定实例ID,就不会显示主机名了
  preferIpAddress: true #访问路径可以显示IP地址 

```

## 启动项目验证1

启动顺序：
9000->9002->8001->8011->8002->8012->7777；
![](https://www.zhangruibin.com/upload/2019/09/qo55eeutjsi7jp787hiotvg11p.png)
此时访问路径：localehost:8012/consumer/product/list,
以及：localehost:7777/CONSUMER-PRODUCT-FEIGN/consumer/product/list

效果一样。

此时的Zuul作用是，没有配置路由规则时，访问Zuul，则Zuul会在EurekaServer中找到对应的/CONSUMER-PRODUCT-FEIGN的IP以及端口号，然后将后面的/consumer/product/list进行转发请求。

如果修改项目7777，在入口启动类中加上@EnableZuulProxy，并且在application.yml中追加Zuul配置：
```
zuul:
 routes: povider-product-01: # 路由名称，名称任意，保持所有路由名称唯一
  path: /consumer/product/** # 访问路径,比如電腦端的轉發到一个请求端，手机端的转发到一个请求端
  serviceId: CONSUMER-PRODUCT-FEIGN # 指定服务ID，会自动从Eureka中找到此服务的ip和端口
  stripPrefix: false # 代理转发时去掉前缀，false:代理转发时不去掉前缀 例如:为true时请求 /product/get/list，代理转发到/get/list
```

此时访问7777项目进行消费者访问就变成了这样：localehost:7777/consumer/product/list。如下图所示：
![](https://www.zhangruibin.com/upload/2019/09/1n945roj8kgcrq7mpcoduq6gdi.png)


截止到现在，整个的demo架构图如下图所示：
![](https://www.zhangruibin.com/upload/2019/09/vv9m4j1dp2gorqdicelqh0j4p6.png)

接下来我们说点Zuul的核心，那就是Zuul的过滤器，通过过滤器实现请求过滤，身份校验等。
## Zuul过滤器

### 自定义过滤器

自定义过滤器需要继承 ZuulFilter，ZuulFilter是一个抽象类，需要覆盖它的4个方法，如下：
* filterType：返回字符串代表过滤器的类型，返回值有：
  * pre：在请求路由之前执行
   * route：在请求路由时调用
    * post：请求路由之后调用， 也就是在route和errror过滤器之后调用
     * error：处理请求发生错误时调用
 * filterOrder：此方法返回整型数值，通过此数值来定义过滤器的执行顺序，数字越小优先级越高。
  * shouldFilter：返回Boolean值，判断该过滤器是否执行。返回true表示要执行此过滤器，false不执行。
   * run：过滤器的业务逻辑。
#### 自定义过滤器 LoginFilter
 * 继承 ZuulFilter
  * 在类上添加 @Component 注解。
   * 实现抽象方法
#### 过滤器代码
LoginFilter.java
```
package com.zhrb.springcloud.filter;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

/**
 * @ClassName LoginFilter
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/11 15:06
 * @Version
  */
   @Component 
   public class LoginFilter extends ZuulFilter{
    Logger logger = LoggerFactory.getLogger(getClass());
    @Override
  public String filterType() {
        return "pre"; //请求路由前调用
  }

    @Override
  public int filterOrder() {
        return 1; //int值来定义过滤器的执行顺序，数值越小优先级越高
  }

    @Override
  public boolean shouldFilter() {
        return true; //该过滤器是否执行，true执行，false不执行
  }

    @Override
  public Object run() throws ZuulException {
        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest request = context.getRequest();
        //获取请求参数token的值
  String token = request.getParameter("loginToken=");
        if (token == null) {
            logger.warn("此操作需要先登录系统");
            context.setSendZuulResponse(false);// 拒绝访问
  context.setResponseStatusCode(200);// 设置响应状态码
  try {
                //响应结果
  context.getResponse().getWriter().write("无登录信息，被LoginFilter所拦截，重定向到登录界面！");
            } catch (IOException e) {
                e.printStackTrace();
            }
            return null;
        }
        logger.info("ok");
        return null;
    }
}
```

### 验证过滤器

启动顺序：9000->9002->8001->8011->8002->8012->7777;


地址栏分别输入：
* 不带loginToken
http://localhost:7777/consumer/product/list
![](https://www.zhangruibin.com/upload/2019/09/psnf839p02ilcppp7vflg7lugn.png)
* 带loginToken
http://localhost:7777/consumer/product/list/?loginToken=test

![](https://www.zhangruibin.com/upload/2019/09/7h4ebc3apainqoqpemk29qephs.png)
至此，Zuul网关已经简单入门，下节说springcloud分布式配置中心，敬请期待。