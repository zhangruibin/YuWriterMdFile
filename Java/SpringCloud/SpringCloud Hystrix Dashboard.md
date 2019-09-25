# SpringCloud Hystrix Dashboard

## SpringCloud Hystrix Dashboard是什么
除了上节中提到的熔断机制，Hystrix还提供了准实时的调用监控（Hystrix Dashboard），Hystrix会持续地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。

## SpringCloud Hystrix Dashboard怎么用
### 新建一个dashboard model项目
项目名称：springclouddemo-06-hystrix-dashboard-7000
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

    <artifactId>springclouddemo-06-hystrix-dashboard-7000</artifactId>
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
        <!--导入 hystrix 与 hystrix-dashboard 依赖-->
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <!-- 添加对hystrix断路器的依赖-->
  <dependency>
            <groupId>com.netflix.hystrix</groupId>
            <artifactId>hystrix-core</artifactId>
            <version>1.5.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-hystrix-dashboard</artifactId>
            <version>1.4.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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
#### 新建application.yml文件
application.yml
```
server:
 port: 7000

eureka:
 client: 
   serviceUrl: 
     defaultZone: http://127.0.0.1:9000/eureka/,http://127.0.0.1:9002/eureka/
spring:
 application: 
   name: hystrix-dashboard
```
#### 新建启动类HystrixDashboard_7000
HystrixDashboard_7000.java
```
package com.zhrb.springcloud;

import com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
import org.springframework.context.annotation.Bean;

/**
 * @ClassName HystrixDashboard_7000
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/10 10:47
 * @Version
  */ 
@EnableHystrix
@EnableDiscoveryClient @EnableHystrixDashboard @SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
public class HystrixDashboard_7000 {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboard_7000.class,args);
    }
    @Bean
  public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```


然后修改一个之前的项目，将其纳入监控，例如8011项目：
首先确定8011项目有pom依赖：
```
<!-- hystrix --> <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
<!-- 添加对hystrix断路器的依赖--> <dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-core</artifactId>
    <version>1.5.6</version>
</dependency>
<!--hystrix监控--> <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

然后修改application.yml
增加：
```
# 在被监控服务上添加暴露端点 management:
 endpoints: 
 web: 
 exposure: 
 include: hystrix.stream
```

## 流程原理

1.hystrix让哪个项目纳入监控就将项目增加依赖及配置如8011；
2.然后用户正常访问时，hystrix就可以去ping接口访问路径，然后获取返回值状态等信息，通过访问8011hystrix监控页面可以看到json格式的数据显示；
3.完成上面两步之后，可以再hystrix-dashboard项目中，配置指定的8011hystrix json 格式信息访问路径，然后hystrix-dashboard项目会将json信息，转化成视图仪表盘格式展示。

## 效果

通过上面的流程原理简单介绍，我们此时可以依次来验证效果：
### springclouddemo-06-hystrix-dashboard-7000
先启动 springclouddemo-06-hystrix-dashboard-7000 项目，验证下项目启动访问：http://localhost:7000/hystrix
页面显示如下图所示：
![](https://www.zhangruibin.com/upload/2019/09/4esuciqb7gj7trlf20tuak4c84.png)

### 启动EurekaServer等项目验证

启动顺序：9000->9002->8001->8011
然后多次访问：http://localhost:8011/product/1
然后访问：http://localhost:8011/actuator/hystrix.stream
我们可以看到json型数据如下：
![](https://www.zhangruibin.com/upload/2019/09/rborbqu9q4goaoov4eq80s4dkg.png)

然后我们来将这个json格式数据地址纳入dashboard监控，访问：http://localhost:7000/hystrix
并且填写相关信息如下图所示：
![](https://www.zhangruibin.com/upload/2019/09/5js900vd26in6of79qbi8uo210.png)

第一个输入框填写监控地址: http://localhost:8011/actuator/hystrix.stream

Delay：控制服务器上轮询监控信息的延迟时间，默认为2000毫秒，可以通过配置该属性来降低客
户端的网络和CPU消耗。

Title：可以通过配置该信息来展示更合适的标题，默认会使用具体监控实例的URL

监控结果：
如果没有请求会一直显示 “Loading…”

这时访问一下 http://localhost:8001/product/

然后再多次访问：http://localhost:8011/product/1

在仪表盘界面我们就可以看到图形化监控了，类似如下：
![](https://www.zhangruibin.com/upload/2019/09/q71o2ildoajhroj7kpvh0qhl0u.png)
常见错误
报： Unable to connect to Command Metric Stream
解决方式：
看被监控服务中是否添加 spring-cloud-starter-netflix-hystrix 与spring-boot-starter-actuato依赖，并且暴露的端
点(要纳入监控的端点)是否有配置： management.endpoints.web.exposure.include=hystrix.stream
后记：一定要注意springboot及springcloud的版本，之前博主最初用hystrix-dashboard的时候路径填写没有/actuator。

后续还会有其他组件的使用入门介绍，有兴趣的小伙伴可以关注公众号的推送哦~







