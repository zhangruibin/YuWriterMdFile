# SpringCloud基础项目搭建

<blockquote>
欢迎关注微信公众号：<a href="" rel="nofollow" target="_blank">程序员小圈圈</a><br>原文首发于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">www.zhangruibin.com</a><br> 本文出自于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">RebornChang的博客</a><br>
转载请标明出处^_^<br>
</blockquote>

在之前的文章中我们介绍了什么是微服务，那么从这篇开始我会一步一步搭出来一个可以用于开发的架构demo，如有错误欢迎指正。
另外 本节所用代码公众号回复：sp1，即可获取下载。
# 环境准备
笔者使用的的是idea2016,jdk1.8.2。

## 配置本地settings.xml
在文件 < /settings > 中加入：
```
<!--指定本地仓库路径-->	
<localRepository>D:\javaresource\maven-repository</localRepository>
<!--在 mirrors 标签下 添加阿里云maven私服库-->
<mirrors>
	<mirror>
		<id>alimaven</id> 
 		<mirrorOf>central</mirrorOf>  
 		<name>aliyun maven</name>  
 		<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
	</mirror>	
</mirrors>
<!-- 在 profiles 标签下指定jdk版本 -->
<profile>
<id>jdk-1.8</id>  
<activation>
	<activeByDefault>true</activeByDefault>
	<jdk>1.8</jdk>
</activation>
<properties>
	<maven.compiler.source>1.8</maven.compiler.source>
	<maven.compiler.target>1.8</maven.compiler.target>
	<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
</properties>
</profile>	
</profiles>

```

# 新建项目

## 新建项目 springcloud-demo-01
新建一个empty项目，保留pom，删除src及其子目录文件，作为主目录工程。
![](https://www.zhangruibin.com/upload/2019/09/on1u9la1oaj91rh4hi0gcemipa.png)
![](https://www.zhangruibin.com/upload/2019/09/on3qpbfva2gfupp0n3bkd2llub.png)
![](https://www.zhangruibin.com/upload/2019/09/4qhh9qidisj3ur6l9c53drdpr4.png)
然后再idea中配置maven环境，指定jdk版本以及settings文件。

配置好maven环境之后，修改项目的pom.xml文件。

笔者的pom文件内容如下：
```
<?xml version="1.0" encoding="UTF-8"?> <project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!--项目相关-->
  <description>公众号：程序员小圈圈 版权所有</description>
    <!--<modules>
        <module>springcloud-02-api</module>
        <module>springclouddemo-03-provider-product-8001</module>
    </modules>-->
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.zrb.springcloud</groupId>
    <artifactId>springcloud-demo-01</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--手动指定 pom -->
  <packaging>pom</packaging>

    <!-- spring boot 采用 2.0.7 版本 -->
  <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.7.RELEASE</version>
    </parent>

    <!--依赖版本声明-->
  <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <!-- spring cloud 采用 Finchley.SR2 版本 -->
  <spring-cloud.version>Finchley.SR2</spring-cloud.version>
        <druid.version>1.0.16</druid.version>
        <mysql-connector-java.version>8.0.13</mysql-connector-java.version>
        <mybatis-spring-boot-starter.version>1.3.2</mybatis-spring-boot-starter.version>
        <lombok.version>1.18.0</lombok.version>
    </properties>

    <!--依赖管理-->
  <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <!--maven不支持多继承，使用import来依赖管理配置-->
  <scope>import</scope>
        </dependency>
        <!--导入 mybatis 启动器-->
  <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis-spring-boot-starter.version}</version>
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
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <!--lombok-->
  <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

</project>
```

建好project之后，我们就可以开始创建子项目了。

## 新建公共model springcloud-02-api
新建demo步骤如下：

![](https://www.zhangruibin.com/upload/2019/09/q4r3nr6lnkijpq9hjp2fp9d6ad.png)
![](https://www.zhangruibin.com/upload/2019/09/37v9rk6bpiio8qo0rp5au2uv8a.png)
![](https://www.zhangruibin.com/upload/2019/09/qmbeb0h0c4hv5ot0vaverp6bvv.png)
![]()

建好之后，父级pom文件会有所改变，引入了新建的model。

新建的springcloud-02-api pom文件如下：
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

    <artifactId>springcloud-02-api</artifactId>

</project>
```

然后我们在api项目新建一个实体类，Product.java

```
package com.zhrb.springcloud.entity;

import lombok.Data;
import lombok.ToString;

/**
 * @ClassName Product
 * @Description TODO
  * @Author zhrb
 * @Date 2019/9/3 14:26
 * @Version
  */ @Data @ToString public class Product {

    //主键
  private Long pid;

    //产品名称
  private String productName;

    // 来自哪个数据库，因为微服务架构可以一个服务对应一个数据库，同一个信息被存储到不同数据库
  private String dbSource;
}
```


api项目就暂时到这里，然后新建第二个model项目：springclouddemo-03-provider-product-8001

## 新建model  springclouddemo-03-provider-product-8001
新建步骤参考api项目，此处就不再赘述了,同样的会改变父级pom，就是笔者注释的< models >里面的内容。

说下pom及其他文件：

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
            <version>1.0-SNAPSHOT</version>
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
</project>
```

## 新建application.yml文件

因为我们不是使用Spring Initializr的形式新建的model，所以yml文件还有启动类需要我们自己去新建。

那么在resources文件夹下新建
application.yml
```
#指定服务端口号
server:
 port: 8001

mybatis:
 config-location: classpath:mybatis/mybatis.cfg.xml # mybatis配置文件所在路径
  type-aliases-package: com.zhrb.springcloud.entity # 所有Entity别名类所在包
  mapper-locations: classpath:mybatis/mapper/**/*.xml # mapper映射文件   spring:
 application: name: springcloud-product #这个很重要，这在以后的服务与服务之间相互调用一般都是根据这个name    datasource:
 type: com.alibaba.druid.pool.DruidDataSource # 当前数据源操作类型
  driver-class-name: com.mysql.cj.jdbc.Driver # mysql驱动包
  url: jdbc:mysql://127.0.0.1:3306/springcloud_db01?serverTimezone=GMT%2B8 # 数据库名称
  username: root
    password: root
    dbcp2:
 min-idle: 5 # 数据库连接池的最小维持连接数
  initial-size: 5 # 初始化连接数
  max-total: 5 # 最大连接数
  max-wait-millis: 150 # 等待连接获取的最大超时时间
```
然后先看下总的项目目录结构图：

![](https://www.zhangruibin.com/upload/2019/09/t5rf92dq5sh3mp0hcr3t69kdq2.png)
各文件及内容如下：

ProductController.java
```
package com.zhrb.springcloud.controller;

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
        return productService.list();
    }
}
```

ProductMapper.java

```
package com.zhrb.springcloud.mapper;

import com.zhrb.springcloud.entity.Product;

import java.util.List;

/**
 * @ClassName ProductMapper
 * @Description TODO
  * @Author Administrator
 * @Date 2019/9/3 14:55
 * @Version
  */   public interface ProductMapper {
    Product findById(Long pid);
    List<Product> findAll();
    boolean addProduct(Product product);
}
```

ProductServiceImpl.java
```
package com.zhrb.springcloud.service.impl;

import com.zhrb.springcloud.entity.Product;
import com.zhrb.springcloud.mapper.ProductMapper;
import com.zhrb.springcloud.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @ClassName ProductServiceImpl
 * @Description TODO
  * @Author Administrator
 * @Date 2019/9/3 15:16
 * @Version
  */   @Service public class ProductServiceImpl implements ProductService{

    @Autowired
  private ProductMapper productMapper;

    @Override
  public boolean add(Product product) {
        return productMapper.addProduct(product);
    }

    @Override
  public Product get(Long id) {
        return productMapper.findById(id);
    }

    @Override
  public List<Product> list() {
        return productMapper.findAll();
    }
}
```
ProductService.java
```
package com.zhrb.springcloud.service;

import com.zhrb.springcloud.entity.Product;

import java.util.List;

/**
 * @ClassName ProductService
 * @Description TODO
  * @Author Administrator
 * @Date 2019/9/3 15:15
 * @Version
  */ public interface ProductService {
    boolean add(Product product);
    Product get(Long id);
    List<Product> list();
}
```
Swagger2Config.java
```
package com.zhrb.springcloud.swagger;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.ParameterBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.schema.ModelRef;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Parameter;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.ArrayList;
import java.util.List;

/**
 * @ClassName Swagger2Config
 * @Description TODO
 * @Author Administrator
 * @Date 2019/9/3 16:43
 * @Version
  */ @Configuration @EnableSwagger2 public class Swagger2Config {
    @Bean
  public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .globalOperationParameters(setHeaderToken())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.zhrb.springcloud.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    private List<Parameter> setHeaderToken() {
        ParameterBuilder tokenPar = new ParameterBuilder();
        List<Parameter> pars = new ArrayList<>();
        tokenPar.name("Authorization").description("令牌").modelRef(new ModelRef("string")).parameterType("header").required(false).build();
        pars.add(tokenPar.build());
        return pars;
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("微服务Swagger调试页面")
                .description("描述：Spring Boot中使用Swagger2构建RESTful APIs")
                .termsOfServiceUrl("https://www.zhangruibin.com/")
                .contact("程序员小圈圈")
                .version("1.0")
                .build();
    }
}
```
SpringCloudProductProvider_8001.java
```
package com.zhrb.springcloud;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.AsyncConfigurerSupport;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.annotation.EnableScheduling;

/**
 * @ClassName ProductProvider_8001
 * @Description TODO
  * @Author Administrator
 * @Date 2019/9/3 15:20
 * @Version
  */   //扫描所有Mapper接口   @EnableScheduling @SpringBootApplication public class SpringCloudProductProvider_8001{
    public static void main(String[] args) {
        SpringApplication.run(SpringCloudProductProvider_8001.class, args);
    }
}
```
## 数据库
新建过以上文件，接下来我们来新建数据库，笔者使用的是Navicat新建的，截图如下：
![](https://www.zhangruibin.com/upload/2019/09/rl238vjac0idar3o0hoqrsatpa.png)
新建好数据库之后，创建表：
```
DROP DATABASE IF EXISTS springcloud_db01;
CREATE DATABASE springcloud_db01 CHARACTER SET UTF8;
USE springcloud_db01;
CREATE TABLE product
(
pid BIGINT NOT NULL PRIMARY KEY AUTO_INCREMENT,
product_name VARCHAR(50),
db_source VARCHAR(50)
);
INSERT INTO product(product_name,db_source) VALUES('手机',DATABASE());
INSERT INTO product(product_name,db_source) VALUES('冰箱',DATABASE());
INSERT INTO product(product_name,db_source) VALUES('电脑',DATABASE());
INSERT INTO product(product_name,db_source) VALUES('洗衣机',DATABASE());
INSERT INTO product(product_name,db_source) VALUES('电视',DATABASE());
INSERT INTO product(product_name,db_source) VALUES('音响',DATABASE());
SELECT * FROM product;
```
## 启动项目并测试

启动过项目之后，访问
http://localhost:8001/swagger-ui.html
即可进入swagger主界面，可以清楚的看到项目接口等信息，截图如下：
![](https://www.zhangruibin.com/upload/2019/09/69gfhefcnkhbaqa62koo5bkb4g.png)
访问：
http://localhost:8001/3
或者
http://localhost:8001/list
页面返回数据如下：
![](https://www.zhangruibin.com/upload/2019/09/up624en1raj25rbug4r94njse9.png)
证明项目基本配置启动没问题，加下来我们会在这基础上进行拓展，因篇幅所限这篇就先到这，如有兴趣请静待后续公众号推送，或者访问笔者博客，谢谢。

**亲**，博主的微信公众号   
 ‘**程序员小圈圈**’开始持续更新了哟~~   
 识别二维码或者直接搜索名字  ‘**程序员小圈圈**’   即可关注本公众号哟~~   
 **不只是有技术哟~~** 
  **还有很多的学习娱乐资源免费下载哦~~** 
  还可以学下教育知识以及消遣娱乐哟~~
  **求关注哟~~**     ![](https://www.zhangruibin.com/upload/2019/08/6re5uggg4khmtrrit534ubh759.png)'

