# 那些年我把MySQL从5.7升到8.0.16
# 为什么要升级
笔者之前linux本地安装的MySQL5.7，但是由于服务器内存较小等原因，时不时的数据库就崩了，蓝瘦，而且早就将redis等服务应用到了docker中，而且想试试MySQL8的新特性，所以想着切换成MySQL8，然后就直接docker安装了，省的本地维护。
# 升级前的准备

## 准备MySQL8.0.16环境
### docker安装MySQL8.0.16
#### 拉取镜像
```
$ docker pull mysql:8.0.16
```
#### 配置本地文件用于挂载到镜像
将全部的配置文件和关联的文件夹统一放到`/opt/docker-mysql`中
```

$ mkdir -p /opt/docker-mysql/conf.d

$ vim /opt/docker-mysql/config-file.cnf

$ mkdir -p /opt/docker-mysql/var/lib/mysql

```
config-file.cnf内容如下：
```

[mysqld]
# 表名不区分大小写
lower_case_table_names=1 
#server-id=1
datadir=/var/lib/mysql
#socket=/var/lib/mysql/mysqlx.sock
#symbolic-links=0
# sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

```
#### 启动镜像

```
docker run --name mysql \
    --restart=always \
    -p 3306:3306 \
    -v /opt/docker-mysql/conf.d:/etc/mysql/conf.d \
    -v /opt/docker-mysql/var/lib/mysql:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=Aa123456! \
    -d mysql:8.0.16
```
#### 设置可以远程登录
linux本地登录上MySQL环境：
```
$ docker exec -it mysql bash
```
进入到bash环境下，登录MySQL
```
msyql -uroot -pAa123456!
```
##### 查看用户登录及密码等信息
```
>select host,user,plugin,authentication_string from mysql.user;
```
列出来的数据类似如下：
| host      | user             | plugin                | authentication_string                                                  |
|--|--|--|--|
| %         | root             | caching_sha2_password | *066C39CBB06FD4D073E2EB842453110EE953F9B6                              |
| localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | root             | mysql_native_password | *066C39CBB06FD4D073E2EB842453110EE953F9B6                              |

##### 修改密码更改plugin形式
此时我们看第一条数据，此时的plugin项为caching_sha2_password，如果我们想要使用Navicat，datagrip之类的软件远程链接数据库的话需要修改第一条数据的密码，修方式如下：
```
>ALTER user 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
```
修改之后，一定要刷新权限：
```
>FLUSH PRIVILEGES;
```

修改之后我们再使用select host,user,plugin,authentication_string from mysql.user;
进行查看的时候，发现第一条数据的 plugin方式变成了：mysql_native_password 。
**此时再重新使用客户端软件进行连接就可以连接了。**
注意：host为%，则为不限IP可远程登录，但是安装过8.0.16直接登录，会提示你  authentication_string授权错误，客户端解析不了caching_sha2_password加密方式。
## 数据的备份与导入
### 数据库备份
备份使用的是mysqldump插件。
#### 非docker环境下备份
```
mysqldump -u $username -p$password $database_name > $backup_dir/$database_name.sql

```
#### docker环境下备份
```
docker exec mysql sh -c 'exec mysqldump $database_name -u $username -p$password' > $backup_dir/$database_name.sql

```

### 数据库导入
#### 非docker环境下导入
```
mysql -uroot -p"password" databasename < /app/sel/blog/mysqlbackup/tale_sel-selblog.sql
```
### docker环境下导入
```
	docker exec -i mysql sh -c 'exec mysql -uroot -p"password"' < /app/sel/blog/mysqlbackup/tale_sel-selblog.sql

```

注意：如果是全量备份整个数据库的话，将指定的database_name 换成--all-databases即可。


# 升级后关于项目的改动

## MySQL依赖版本更换
mysql-connector-java版本更改为8.0.16，截止到20191204，没有8.0.16版本的，所以用8.0.X都行。
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.13</version>
</dependency>

## 驱动更换

### 8以下
```
Class.forName("com.mysql.jdbc.Driver"); //MYSQL驱动
```
### 8.0.16
```
Class.forName("com.mysql.cj.jdbc.Driver"); //MYSQL驱动
```

如果驱动没有修改会报如下错误：
```
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.

```

## 数据库连接增加参数
### useSSL

要在数据库连接上明确指定是否启用SSL加密，加密的话肯定是影响数据传输速度的，故笔者直接设置：
useSSL=false。
否则会报如下错误：
```
 Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
```
### 设置时区

```
serverTimezone=GMT%2B8
```


修改后的数据库URL连接例如下所示：
```
jdbc:mysql:jdbc:mysql://localhost:3306/db_name?useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&useSSL=false&serverTimezone=GMT%2B8
```



