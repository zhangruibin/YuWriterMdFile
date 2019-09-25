# Spring Cloud Config

## 为什么要用Spring Cloud Config
在分布式微服务架构中，由于服务数量很多 ，使得有很多配置文件，在更新配置文件时很麻烦。
我们每个微服务自已带着一个 application.yml，上百个配置文件的管理起来就很麻烦，所以一套集中式的、动态的配置管理功能是必不可少的，在Spring Cloud中，有分布式配置中心组件Spring Cloud Config来解决这个问题。
## Spring Cloud Config是什么

Spring Cloud Config 分为 服务端与客户端 两个部分：
服务端 config server：
也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信
息，加密、解密信息等访问接口。
配置服务器官方推荐采用 Git 来存储配置信息，这样就有助于对环境配置进行版本管理，并且可通过Git
客户端工具来方便的管理与访问配置信息。
客户端 config client：
通过指定的服务端来管理服务的资源，以及与业务相关的配置内容，并在启动的时候从服务端获取和加载配置信息.
## Spring Cloud Config有什么用
* 集中管理配置文件
* 不同环境不同配置，动态化的配置更新，根据不同环境部署，如 dev/test/prod
* 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置，服务会向配置中心统一拉取自已的配
置信息
* 当配置发生变动时，服务不需要重启即可感知到配置的变化并使用修改后的配置信息
* 将配置信息以REST接口的形式暴露

下面看下结构图如下图所示：
![](https://www.zhangruibin.com/upload/2019/09/0753290i1ug59pj5v8vepv2n0q.png)

Spring Cloud Config 分为 服务端与客户端 两个部分：
* 服务端 config server：
也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信
息，加密、解密信息等访问接口。
配置服务器官方推荐采用 Git 来存储配置信息，这样就有助于对环境配置进行版本管理，并且可通过Git
客户端工具来方便的管理与访问配置信息。
* 客户端 config client：
通过指定的服务端来管理服务的资源，以及与业务相关的配置内容，并在启动的时候从服务端获取和加
载配置信息。

# 使用步骤
## Spring Cloud Config 服务端
### git配置文件环境准备
**首先本机要安装并配置好Git环境！！**
1. 先在自己的GitHub上新建一个空的public仓库，命名为springcloud-config。

2. Clone with HTTPS
拷贝仓库的https地址（https://github.com/zhangruibin/springcloud-config.git）
然后使用Gitbash或者cmd命令行，拷贝仓库环境到本地。
比如博主是cd到demo项目主目录然后执行命令：
```
git clone https://github.com/zhangruibin/springcloud-config.git
```

3. 在通过clone出来的文件夹中新建配置文件
执行了clone命令后，会在当前目录生成一个目录，目录名为springcloud-config，文件夹中还有.git文件。
拷贝一个项目的application.yml并将文件的名字修改成：springcloud-config-application.yml。
然后修改文件的内容为：
```
#注意：如果在记事本上编写，下面的缩进不要使用Tab来缩进，不然报错 #保存时注意，选择 UTF-8 保存 spring:
 profiles: active: dev # 激活开发环境配置 ---
server:
 port: 4001 #端口号 spring:
 profiles: dev # 开发环境
  application:
 name: springcloud-config-dev
---
server:
 port: 4002 #端口号 spring:
 profiles: prod # 生产环境
  application:
 name: springcloud-config-prod
```

一定要注意编码格式及缩进，还有文件类型为yml而不是TXT。
4. 将文件推送到GitHub仓库
怎样将文件推送到Github仓库笔者之前有写，文章链接：
https://blog.csdn.net/m0_37190495/article/details/79912212

这里就不细说了，命令及截图如下：
![](https://www.zhangruibin.com/upload/2019/09/1k1uvsv71cg7co1p3ukaffnd2l.png)

5. 登录仓库验证推送无误

### 新建项目springclouddemo-08-config-5001

**pom.xml**
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

    <artifactId>springclouddemo-08-config-5001</artifactId>
    <dependencies>

        <!-- Spring Cloud Config配置中心依赖 -->
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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

**application.yml**

```
server:
 port: 5001
spring:
 application: name: springcloud-config
  cloud:
 config: server: git: # 远程库的git地址
  uri: https://github.com/zhangruibin/springcloud-config.git
```


**ConfigServer_5001.java**

```
package com.zhrb.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.cloud.config.server.EnableConfigServer;

/**
 * @ClassName ConfigServer_5001
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/12 11:02
 * @Version
  */ @EnableConfigServer//开启配置中心功能 @SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
public class ConfigServer_5001 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServer_5001.class, args);
    }
}
```

### 启动5001项目验证
springcloud config 的配置文件读取规则如下：
```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

我们用的是yml，所以我们使用第二种访问方式。
启动5001项目并访问以下地址：
1. localhost:5001/springcloud-config-application.yml
2. localhost:5001/springcloud-config-application-dev.yml
3. localhost:5001/springcloud-config-application-prod.yml
效果如下图所示：
![](https://www.zhangruibin.com/upload/2019/09/6jirc58ot8hegpn4g6p6aklvpl.png)
![](https://www.zhangruibin.com/upload/2019/09/0v26ls3a7ij4dqfvc0bmoo6iml.png)
![](https://www.zhangruibin.com/upload/2019/09/5u4pn8tb88g83rak9n5sfkmcns.png)
可以看到可以正确读取文件信息。
此时，服务端我们已经构建完毕，接下来就是客户端了。

## Spring Cloud Config 客服端服务

### 在之期的基础上新建8080项目
新建model项目：springclouddemo-09-config-client-8080。
1. pom.xml
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

    <artifactId>springclouddemo-09-config-client-8080</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- Spring Cloud Config 客户端依赖-->
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>com.zrb.springcloud</groupId>
            <artifactId>springcloud-02-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
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

2. bootstrap.yml
```
spring:
 cloud: config: name: springcloud-config-application #github上的配置名称，注意没有yml后缀名
  profile: dev #本次访问的环境配置项
  label: master #远程库的分支名
  uri: http://localhost:5001 #Config配置中心地址，通过它获取springcloud-config-application.yml配置信息
```

3. application.yml

```
server:
 port: 8080
spring:
 application: name: springcloud-config-client
```

4. ConfigClient.java

```
package com.zhrb.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @ClassName ConfigClient
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/12 11:22
 * @Version
  */ 
@RestController
public class ConfigClient {
    //会从github中的springcloud-config-application.yml中获取
  @Value("${spring.application.name}")
    private String applicationName;
    @Value("${server.port}")
    private String port;
    //请求访问
  @RequestMapping("/config")
    public String getConfig() {
        String content = "applicationName:" + applicationName + ", port:" +
                port;
        System.out.println("content: " + content);
        return content;
    }
}
```

通过注解的方式获取配置文件的值，因为我们现在用的springcloud config的配置中心，所以获取到的application。name，及server.port应该是远端仓库中所配置的。
5. ConfigClient_8080.java
```
package com.zhrb.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

/**
 * @ClassName ConfigClient_8080
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/12 11:27
 * @Version
  */ @SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
public class ConfigClient_8080 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClient_8080.class, args);
    }
}
```

### 启动并验证
启动5001级8080项目。访问以下地址：
1. locaohost:8080/config
![](https://www.zhangruibin.com/upload/2019/09/sthm45mbigjk5p4cbesf14fi5b.png)
2. localhost:4001/config
![](https://www.zhangruibin.com/upload/2019/09/0t4t2j3rbojmvo63ge7ijier4k.png)

从上图我们可以看到，项目读取到的配置文件就是我们推送到github中的配置文件。
至此，springcloud config 已经初步成功，由于篇幅所限，本节到此结束，下节我们说springcloud config 在springcloud 微服务中怎么用的，敬请期待。




