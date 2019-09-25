# SpringCloud Eureka
<blockquote>
欢迎关注微信公众号：<a href="" rel="nofollow" target="_blank">程序员小圈圈</a><br>原文首发于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">www.zhangruibin.com</a><br> 本文出自于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">RebornChang的博客</a><br>
转载请标明出处^_^<br>
</blockquote>
**公众号回复:sp2，即可获取本节代码哦~~**
## 在前文基础上新建模块springclouddemo-04-consumer-product-8002

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

    <artifactId>springclouddemo-04-consumer-product-8002</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.zrb.springcloud</groupId>
            <artifactId>springcloud-02-api</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```
新建后项目目录如下图所示：

![](https://www.zhangruibin.com/upload/2019/09/1fmhc4m3tgifcpaaoom4sjso7q.png)

## 新建目录及文件
新建application.yml 文件，声明模块访问端口为8002.

```
server:
 port: 8002
```
在ConfigBean.java里面，配置RestTemplate.
这里我们使用spring提供的web组件进行接口的请求，来访问8001项目获取商品信息。

ConfigBean.java
```
package com.zhrb.springcloud.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * @ClassName ConfigBean
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/4 11:56
 * @Version
  */ @Configuration public class ConfigBean {
    // 向容器中添加 RestTemplate 组件,直接通过此组件可调用 REST 接口
  @Bean
  public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

ProductControllerConsumer.java
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
  */ 
  @RestController
   @RequestMapping(value = "/consumer/product")
public class ProductControllerConsumer {

    private static final String REST_URL_PREFIX = "http://127.0.0.1:8001";

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

启动类：
SpringCloudProductConsumer_8002.java
```
package com.zhrb.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

/**
 * @ClassName SpringCloudProductConsumer_8002
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/4 11:59
 * @Version
  */ @SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
public class SpringCloudProductConsumer_8002 {
    public static void main(String[] args) {
        SpringApplication.run(SpringCloudProductConsumer_8002.class, args);
    }
}
```

这里有一个小知识点，我们可以看到，8002项目所用注解，和8001项目不一样，注解后面多了一些东西，这是为什么呢？如果不加上那些会有什么错吗？看下面。

根据springboot官方文档解释，@SpringBootApplication是一个复合注解。

简单的说就是：@SpringBootApplication相当于@Configuration,@EnableAutoConfiguration,@ComponentScan。

1、@Configuration的注解类标识这个类可以使用Spring IoC容器作为bean定义的来源。@Bean注解告诉Spring，一个带有@Bean的注解方法将返回一个对象，该对象应该被注册为在Spring应用程序上下文中的bean。

2、@EnableAutoConfiguration：能够自动配置spring的上下文，试图猜测和配置你想要的bean类，通常会自动根据你的类路径和你的bean定义自动配置。

3、@ComponentScan：会自动扫描指定包下的全部标有@Component的类，并注册成bean，当然包括@Component下的子注解@Service,@Repository,@Controller。

所以，在启动类上使用@SpringBootApplication时，会自动根据启动类路径进行扫描，执行了@ComponentScan，因此，在没有配置mapper时，一定要使用@DataSourceAutoConfiguration排除默认扫描配置。否则启动报错。

### 启动8002验证
先启动8001，再启动8002.
访问地址：localhost:8002/consumer/product/list
效果如下图所示：
![](https://www.zhangruibin.com/upload/2019/09/q7anepi0sohv8pfd0ss2h3u82j.png)
新建完上面的那些项目之后，就迎来了我们接触springcloud的第一个组件，eureka。
## Eureka
### Eureka介绍
Spring Cloud Eureka 是对Netflix公司的Eureka的二次封装，它实现了服务治理的功能，Spring Cloud Eureka 提供 Eureka Server 服务端与 Eureka Client 客户端 ，服务端即是Eureka服务注册中心，客户端完成微服务向Eureka服务的注册与发现。

客户端同时也具备一个内置的使用轮询(round-robin)负载算法的负载均衡器。在微服务启动后，将会向
Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心
跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）。

1. Eureka Server 是服务端，负责管理各个微服务注册和发现。
2. 在微服务上添加 Eureka Client 代码，就会访问到 Eureka Server 将此微服务注册在Eureka Server中，从而
使服务消费方能够找到。
3. 微服务（服务消费者）需要调用另一个微服务（服务提供者）时，从 Eureka Server 中获取服务调用地址，
进行远程调用。


来看一下服务于注册Eureka架构图

![](https://www.zhangruibin.com/upload/2019/09/582mqk8mc2h0krc6drn4dlu0ju.png)

看完了上面，是不是很像dubboo+zookeeper的服务注册与发现？
那么我们现在再新建一个类似dubbo-admin的模块，来进行页面视图的Eureka服务控制台。

### Eureka 服务中心
新建模块springclouddemo-05-eureka-9000

新建后只需要新建两个文件：
1.application.yml

```
server:
 port: 9000 # 服务端口   eureka:
 instance: hostname: eureka-server # eureka服务端的实例名称
  client:
 registerWithEureka: false # 服务注册，false表示不将自已注册到Eureka服务中
  fetchRegistry: false # 服务发现，false表示自己不从Eureka服务中获取注册信息
  serviceUrl: # Eureka客户端与Eureka服务端的交互地址，集群版配置对方的地址，单机版配置自己（如果不配置则默认本机8761端口）
  defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

关于yml文件，一定一定要注意缩进问题，否则会编译报错！

2.启动类EurekaServer_9000

```
package com.zhrb.springcloud;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * @ClassName EurekaServer_9000
 * @Description TODO
  * @Author Administrator
 * @Date 2019/9/5 9:02
 * @Version
  */ 
  @RestController @EnableEurekaServer @SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
public class EurekaServer_9000 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer_9000.class, args);
    }
    @Value("${server.port}")
    String port;
    @RequestMapping("/hi")
    public String home(@RequestParam String name) {
        return "hi "+name+",i am from port:" +port+"this is my first test eureka";
    }
    @RequestMapping("/shishi")
    public Object getNumber(){
        String s = "";
        for (int i=1;i<=9;i++){
            for (int j = 1;j<=i;j++){
                s+=i+"*"+j+"="+(i*j)+"\t";
                System.out.print(i+"*"+j+"="+(i*j)+"\t");
            }
            System.out.print("\n");
        }
        return s;
    }
}
```

然后就是最重要的pom文件：

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

    <artifactId>springclouddemo-05-eureka-9000</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
            <version>1.3.5.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <!--热部署-->
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
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

然后启动项目后截图如下：
![]()
可以看到，此时是没有项目向注册中心注册的。
接下来改造8001，8002向注册中心注册。

### 改造8001，8002
改造两个项目，让两个项目向注册中心注册：
![](https://www.zhangruibin.com/upload/2019/09/s9e0e9tbu2jt2oool6vmfj8vc8.png)
##### 8001
改造后的pom如下：
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
    <properties>
        <swagger.version>2.6.1</swagger.version>
        <swagger.ui.version>2.6.1</swagger.ui.version>
    </properties>
    <artifactId>springcloud-demo-03-provider-product-8001</artifactId>
    <dependencies>
        <!--springboot web启动器-->
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- mybatis 启动器-->
  <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>

        <dependency>
            <groupId>com.zrb.springcloud</groupId>
            <artifactId>springcloud-02-api</artifactId>
            <version>${project.version}</version>
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
        <!--druid数据源-->
  <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${druid.version}</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql-connector-java.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.0</version>
        </dependency>
        <!--导入 mybatis 启动器-->
  <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis-spring-boot-starter.version}</version>
        </dependency>
        <!--lombok bean 生成辅助框架-->
  <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
        </dependency>
        <!--swagger 前后端分离调试用-->
  <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>${swagger.version}</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>${swagger.ui.version}</version>
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

application.yml如下：
```
server:
 port: 8001

mybatis:
 config-location: classpath:mybatis/mybatis.cfg.xml # mybatis配置文件所在路径
  type-aliases-package: com.zhrb.springcloud.entity # 所有Entity别名类所在包
  mapper-locations: classpath:mybatis/mapper/**/*.xml # mapper映射文件   spring:
 application: name: springcloud-product #这个很重要，这在以后的服务与服务之间相互调用一般都是根据这个name    datasource:
 type: com.alibaba.druid.pool.DruidDataSource # 当前数据源操作类型
  driver-class-name: com.mysql.cj.jdbc.Driver # mysql驱动包
  url: jdbc:mysql://104.225.147.76:3306/springcloud_db01?serverTimezone=GMT%2B8 # 数据库名称
  username: root
    password: Zhrb1992!
    dbcp2:
 min-idle: 5 # 数据库连接池的最小维持连接数
  initial-size: 5 # 初始化连接数
  max-total: 5 # 最大连接数
  max-wait-millis: 150 # 等待连接获取的最大超时时间 eureka:
 client: serviceUrl: defaultZone: http://localhost:9000/eureka/
```

然后在启动类上增加@EnableEurekaClient 即可。

#### 改造8002

pom
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
    <artifactId>springclouddemo-04-consumer-product-8002</artifactId>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Dalston.SR4</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.zrb.springcloud</groupId>
            <artifactId>springcloud-02-api</artifactId>
            <version>${project.version}</version>
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
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <!--热部署-->
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
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


application.yml

```
server:
 port: 8002

eureka:
 client: serviceUrl: defaultZone: http://localhost:9000/eureka/
spring:
 application: name: consumer-product
```
然后在启动类上增加@EnableEurekaClient 即可。

注意，后面的spring-boot-maven-plugin，以及pring-cloud-dependencies是必须的，否则在client找不到指定的版本。


### 启动项目验证
注意项目的启动顺序，先启动服务中心，再启动其他服务注册。
即：9000->8001->8002
![](https://www.zhangruibin.com/upload/2019/09/59741sbssajkcojed2a64lsbe1.png)
启动后可以看到两个项目都注册到了服务中心。

这里有一点，当我其中一个项目宕机时，注册中心依然能看到这个服务，这不太好啊，那怎么办呢？别急，后续的springcloud组件中有关于这点的处理，那就是服务的熔断机制。

 **亲**，博主的微信公众号   
 ‘**程序员小圈圈**’开始持续更新了哟~~   
 识别二维码或者直接搜索名字  ‘**程序员小圈圈**’   即可关注本公众号哟~~   
 **不只是有技术哟~~** 
  **还有很多的学习娱乐资源免费下载哦~~** 
  还可以学下教育知识以及消遣娱乐哟~~
  **求关注哟~~**     ![](https://www.zhangruibin.com/upload/2019/08/6re5uggg4khmtrrit534ubh759.png)'



