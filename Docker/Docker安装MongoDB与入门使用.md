# Docker安装MongoDB与入门使用
## 安装MongoDB
1.查看可用镜像
首先我们瞅一眼现有的docker中MongoDB版本：
docker search mongo。
发现有latest版本，那么我们直接pull下来用就行了：
```
$ docker pull mongo:latest
```

2.pull镜像
pull结束之后来看下是否成功pull到本地了：

```
$ docker images |grep mongo
```

3.运行镜像
如笔者，可以看到已经有本地容器 中已经有MongoDB镜像了，那么我们来运行镜像吧：

```
 docker run -itd --name mongo -p 27017:27017 mongo --auth
```

参数说明：

*   -p 27017:27017 ：映射容器服务的 27017 端口到宿主机的 27017 端口。外部可以直接通过 宿主机 ip:27017 访问到 mongo 的服务。
*   --auth：需要密码才能访问容器服务。

运行成功的话会返回一个容器运行id，此时我们使用命令：
docker ps -a 可以看到运行成功的实例，状态为up。
4.赋予用户
然后我们为该实例增加账户密码：
$ docker exec  -it mongo mongo admin # 创建一个名为 admin，密码为 123456 的用户。  > db.createUser({ user:'admin',pwd:'123456',roles:[  { role:'userAdminAnyDatabase', db:  'admin'}]});  # 尝试使用上面创建的用户信息进行连接。  > db.auth('admin',  '123456')


看到输出结果为1，则证明现在已经创建好用户了。
## MongoDB的简单使用

关于MongoDB是什么这里就不怎么说了，毕竟这篇不是概念科普，我们如果笼统的说，那就把它当成异形redis吧。

那么redis有五种数据结构(截止到现在应该是6种，最新版增加了stream类型)，MongoDB有几种呢？
### MongoDB数据类型
| 数据类型 | 描述 |
| -------- | --------- |
| String | 字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。 |
| Integer | 整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。 |
| Boolean | 布尔值。用于存储布尔值（真/假）。 |
| Double | 双精度浮点值。用于存储浮点值。 |
| Min/Max keys | 将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。 |
| Array | 用于将数组或列表或多个值存储为一个键。 |
| Timestamp | 时间戳。记录文档修改或添加的具体时间。 |
| Object | 用于内嵌文档。 |
| Null | 用于创建空值。 |
| Symbol | 符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。 |
| Date | 日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。 |
| Object ID | 对象 ID。用于创建文档的 ID。 |
| Binary Data | 二进制数据。用于存储二进制数据。 |
| Code | 代码类型。用于在文档中存储 JavaScript 代码。 |
| Regular expression | 正则表达式类型。用于存储正则表达式。 |
### MongoDB术语解释
| SQL术语 | MongoDB术语 | 解释/说明 |
| -- | -- | -- |
| database | database | 数据库 |
| table | collection | 数据库表/集合 |
| row | document | 数据记录行/文档 |
| column | field | 数据字段/域 |
| index | index | 索引 |
| table joins |   | 表连接,MongoDB不支持 |
| primary key | primary key | 主键,MongoDB自动将_id字段设置为主键 |
