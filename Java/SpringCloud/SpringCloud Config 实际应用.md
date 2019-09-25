# SpringCloud Config 实际应用


公众号回复：sp7，即可获取截至本节的代码哦~

上节中我们说了基本的springcloud config的使用方法，这节中我们说下在实际应用中的springcloud config是怎么使用的。


首先我们在上节的基础上新建两个文件，一个文件是Eurekaserver的配置文件，一个是类似之前8001及8011项目的配置文件：
* springcloud-config-application-eureka.yml
```
spring:
 profiles: active: dev # 激活开发环境配置 ---
server:
 port: 6001 #端口号 spring:
 profiles: dev # 开发环境
  application:
 name: springcloud-config-eureka
eureka:
 instance: hostname: eureka-server
  client:
 registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
 defaultZone: http://localhost:6001/eureka/
  server:
 enable-self-preservation: false # 禁用自我保护机制***************** ---
server:
 port: 6001 #端口号 spring:
 profiles: prod # 生产环境
  application:
 name: springcloud-config-eureka
eureka:
 instance: hostname: eureka-server
  client:
 registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
 defaultZone: http://localhost:6001/eureka/
  server:
 enable-self-preservation: true # 开启自我保护机制*****************
```



* springcloud-config-product.yml
```
#注意：如果在记事本上编写，下面的缩进不要使用Tab来缩进，不然报错 spring:
 profiles: active: prod # 激活开发环境配置 ---
server:
 port: 8001
mybatis:
 config-location: classpath:mybatis/mybatis.cfg.xml
  type-aliases-package: com.zhrb.springcloud.entity
  mapper-locations: classpath:mybatis/mapper/**/*.xml
spring:
 profiles: dev # 开发环境
  application:
 name: springcloud-product-config # ******服务名*******
  datasource:
 type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://104.225.147.76:3306/springcloud_db01?serverTimezone=GMT%2B8 #数据库连接地址，*****注意库名 db01***************
  username: cxyxqq
    password: Cxyxqq123!
    dbcp2:
 min-idle: 5
      initial-size: 5
      max-total: 5
      max-wait-millis: 150
eureka:
 client: serviceUrl: defaultZone: http://127.0.0.1:6001/eureka/
---
server:
 port: 8888
mybatis:
 config-location: classpath:mybatis/mybatis.cfg.xml
  type-aliases-package: com.zhrb.springcloud.entity
  mapper-locations: classpath:mybatis/mapper/**/*.xml
spring:
 profiles: prod # 生产环境
  application:
 name: springcloud-product-config # ******服务名*******
  datasource:
 type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://104.225.147.76:3306/springcloud_db02?serverTimezone=GMT%2B8 #数据库连接地址，*****注意库名 db02***************
  username: cxyxqq
    password: Cxyxqq123!
    dbcp2:
 min-idle: 5
      initial-size: 5
      max-total: 5
      max-wait-millis: 150
eureka:
 client: serviceUrl: defaultZone: http://127.0.0.1:6001/eureka/
```

可以看到，我们将开发环境的端口定义成8001，数据库为db01。将生产环境的端口定义成8888，数据库为db02。
并且他们的eurekaserver是6001。

接下来我们就新建模块来模拟springcloud config在实际中的使用。

## 新建模块

### 新建model springclouddemo-10-eureka-config-6001
* pom.xml

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

    <artifactId>springclouddemo-10-eureka-config-6001</artifactId>

    <dependencies>
        <!--springboot web启动器-->
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--需要引入该jar才能使bootstrap配置文件生效-->
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-context</artifactId>
        </dependency>
        <!-- Spring Cloud Config 客户端依赖-->
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <!-- 导入Eureka-server 服务端依赖 -->
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
            <version>1.3.5.RELEASE</version>
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


* bootstrap.yml

```
spring:
 cloud: config: name: springcloud-config-application-eureka #github上的配置名称，注意没有yml后缀名
  profile: dev #本次访问的环境配置项
  label: master #远程库的分支名
  uri: http://localhost:5001 #Config配置中心地址，通过它获取microservice-configeureka.yml配置信息
```


* application.yml

```
server:
 port: 6001
spring:
 application: name: springcloud-eureka-config
```

* 启动类EurekaServer_Config_6001.java

```
package com.zhrb.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

/**
 * @ClassName EurekaServer_Config_6001
 * @Description TODO
  * @Author Administrator
 * @Date 2019/9/12 16:18
 * @Version
  */ @EnableEurekaServer @SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
public class EurekaServer_Config_6001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer_Config_6001.class, args);
    }
}
```

### 新建服务提供者springclouddemo-11-provider-product-config-8001

复制springclouddemo-03-provider-product-8001并修改相关信息为springclouddemo-11-provider-product-config-8001，并添加依赖到parent。
pom文件增加依赖：
```
<!--需要引入该jar才能使bootstrap配置文件生效--> <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-context</artifactId>
</dependency>
<!-- Spring Cloud Config 客户端依赖--> <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```


* application.yml文件内容修改为

```
spring:
 application: 
   name: springcloud-product-config
```

* 新建bootstrap.yml
```
spring:
 cloud: config: name: springcloud-config-product #github上的配置名称，注意没有yml后缀名
  profile: prod # 本次访问的环境配置项
  label: master # 远程库的分支名
  uri: http://localhost:5001 #Config配置中心地址，通过它获取microservice-config-product.yml配置信息
```


做完上面这些，我们来大致梳理下整个业务的执行流程：

环境设置为dev的时候，用户访问的应该是8001端口，这时我们将项目中的bootstrap.yml文件中的profile设置为dev.GitHub中的spring active也为dev.

当启动6001（eurekaserver）->5001（配置中心）->新8001时，8001项目会根据bootstrap.yml文件中的配置去5001项目关联的GitHub库找对应的文件配置来作为自己的配置。

当用户请求8001端口的请求时，会访问我们的新建model项目，然后根据从GitHub中获取的配置来进行环境设置及逻辑访问。
大致流程如下图所示：

![](https://www.zhangruibin.com/upload/2019/09/1qrheo2djkin6pe3vcmf5sif2q.png)
那么接下来我们启动验证下是不是这样。

## 启动项目验证效果

6001（eurekaserver）->5001（配置中心）->新8001。

* 访问：localhost:6001/eureka/
截图如下：
![](https://www.zhangruibin.com/upload/2019/09/4o61jh9j08jjgq4gm0grqbklb6.png)
证明读取配置文件成功，并且新8001项目向6001服务中心进行了注册。

* 当环境设置为dev的时候，访问：
localhost:8001/product/1
效果如下:
![](https://www.zhangruibin.com/upload/2019/09/7cme8bmuosietq40ihl615pusk.png)
* 当环境设置为prod的时候,**重新启动新8001项目**，访问：
localhost:8001/product/1
效果如下:
![](https://www.zhangruibin.com/upload/2019/09/r9767lmu5eiq7ovos5a0arqhap.png)
* 当环境设置为prod的时候，访问：
localhost:8888/product/list
效果如下:

![](https://www.zhangruibin.com/upload/2019/09/1dfsaa5amejgbpvq8o7vs6gms1.png)
从上面的截图中我们可以看到使用配置中的好处：
操作简单，修改配置文件不用启动项目，并且更换环境也简单了许多，文件集中管理更方便配置。
当然缺点也有，那就是要有专门的库去进行配置文件的维护，维护成本较不使用略高。

而且细心的小伙伴有没有发现，在切换环境的时候我着重写了**重启新8001项目**，那不重启行不行，我改点配置文件就得重启项目那成本有点大了，没关系，看下节的解决办法。


讲到这里，一代版的springcloud就剩一节了，那就是springcloud bus。
细心的小伙伴可能发现了，我上面说的是一代版，那是不是有二代版呢？有的，之后会对一二代的区别及使用进行介绍，敬请期待。



