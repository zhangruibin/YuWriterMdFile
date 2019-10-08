# 使用nginx配置HTTPS


看本篇的少年哟，笔者默认你安装好了nginx，并且做了端口转发哟，如果没做的话可以翻看笔者博客网站:https://zhangruibin.com，上面有文章讲述哦，那么接下来就是说配置https证书这点事了。
## 什么是https
首先我们说下什么是https(没错，这说明是笔者用来凑字数的~)。
HTTPS（全称：Hyper Text Transfer Protocol over Secure Socket Layer 或 Hypertext Transfer Protocol Secure，超文本传输安全协议），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。 它是一个URI scheme（抽象标识符体系），句法类同[http](https://baike.baidu.com/item/http):体系。用于安全的HTTP数据传输。https:URL表明它使用了HTTP，但HTTPS存在不同于HTTP的默认[端口](https://baike.baidu.com/item/%E7%AB%AF%E5%8F%A3/103505)及一个加密/身份验证层（在HTTP与TCP之间）。这个系统的最初研发由[网景](https://baike.baidu.com/item/%E7%BD%91%E6%99%AF)公司(Netscape)进行，并内置于其浏览器Netscape Navigator中，提供了身份验证与加密[通讯](https://baike.baidu.com/item/%E9%80%9A%E8%AE%AF/396194)方法。现在它被广泛用于[万维网](https://baike.baidu.com/item/%E4%B8%87%E7%BB%B4%E7%BD%91)上安全敏感的通讯，例如交易支付方面。

上面那段来自百度百科~

## 怎么做https

那知道了什么是HTTPs我们就来做下https吧，常见的httpz转https技术分为两种，一种是将证书配置到Tomcat上，一种是将证书配置在nginx上，笔者推荐配置在nginx上，这样升级证书的时候可以做到批量升级，比如笔者的两个域名。一个https://zhangruibin.com  一个 https://92cnb.com

升级证书的时候只需要将nginx停掉更换证书就行了。

那么接下来简介下怎么nginx怎么配置https：


因为nginx的https是需要依赖ssl模块的，但是nginx默认的模块并没有ssl，所以要我们手动安装编译：
使用nginx -V 就可以看到nginx的模块，如笔者这样：

```
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

可以看到有http_ssl_module 那就是安装了ssl模块，如果没有的话安装模块也很简单，这里就不详细说在linux下安装nginx了，毕竟篇幅有限，网上也有很多资源。
# nginx配置https

## 安装ssl模块


下载 nginx 安装包, nginx官网1.14.1稳定版本tar.gz包。
### 下载安装包到 src 目录
$ cd /usr/local/src
$ wget http://nginx.org/download/nginx-1.14.1.tar.gz

解压安装包。
$ tar -zxvf nginx-1.14.1.tar.gz

配置 ssl 模块。
$ cd nginx-1.14.1
$ ./configure --prefix=/usr/local/nginx --with-http_ssl_module

使用 make 命令编译（使用make install会重新安装nginx），此时当前目录会出现 objs 文件夹。
用新的 nginx 文件覆盖当前的 nginx 文件。
$ cp ./objs/nginx /usr/local/nginx/sbin/

再次查看安装的模块，可以看到ssl模块已经安装了。


那么我们接下来说下https证书那点事。

## https ssl证书那点事

简单的说ssl证书分为三种：DV SSl，OV SSL，EV SSL。
### DV SSl
 适用对象：中小型企业网站、中小型电子商务网站、电子邮局服务器、个人网站等。

### OV SSL
适合对象：企业网站、电子商务网站、证券、金融机构。

### EV SSL
EV SSL 证书具有最高级别可信度及安全性，从而使访问者更加确信以及更加放心的相信他们所进行交易的网站是真实合法的，提升在线交易量。中国证书 提供的 EV SSL证书，其证书颁发机构(CA)均通过WebTrust认证。

那详细的呢？这里就不占篇幅了，看百度吧少年：
https://jingyan.baidu.com/article/ac6a9a5e28ed872b643eac4c.html


所以看了上面的介绍，我们就明白了，笔者作为个人网站使用的，那就一点最重要：免费！
所以只能选择DV了，接下来本篇的重点就来了，就是快速生成DV证书。


## 快速生成DV证书

这里我们要克隆一个git上的仓库，人家仓库有现成的方法，我们只需要使用命令调用就行了。

### 使用Git clone仓库
在linux主目录下：
$ cd /
$ git clone https://github.com/letsencrypt/letsencrypt
$ cd letsencrypt
$ ./certbot-auto --help all

### 获取签名
以笔者的小网站为例：

openssl req -new -newkey rsa:2048 -sha256 -nodes -out example_com.csr -keyout example_com.key -subj "/C=CN/ST=BeiJing/L=Beijing/O=BixinTec Inc./OU=Web Security/CN=zhangruibin.com"


注意，获取签名的时候需要使用80端口，如果此时nginx占用了80端口，要先将nginx停掉，否则会获取失败。

成功的话会生成两个文件：
-rw-r--r-- 1 root root 1029 Jul  8 04:05 example_com.csr
-rw-r--r-- 1 root root 1708 Jul  8 04:05 example_com.key

这两个我们不予理会，继续。

### 根据域名生成证书

./letsencrypt-auto certonly --standalone --email itbin@foxmail.com -d zhangruibin.com -d www.zhangruibin.com


### 证书输出的文件
 Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/92cnb.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/92cnb.com/privkey.pem
   
  Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/zhangruibin.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/zhangruibin.com/privkey.pem

截止到这里DV证书就生成成功了，然后我们将其放在nginx的conf目录，在conf目录新建一个文件夹专门存放这两个文件，比如博主新建的是tecssl。
值得一提的是，这种方式生成的DV证书有效期是三个月，三个月到期只需要再次获取签名生成证书再替换就行了。

可以写个脚本，笔者有时间写下然后再贴出来供大家参考。

配置完成后登陆网站：https://zhangruibin.com可以看到证书效果如下：
![](https://www.zhangruibin.com/upload/2019/10/7hg0po5mkmj3tr90qg60jgtksa.png)

附录，贴下nginx配置https的那块配置：
```
 
    server {
        listen       80;
        listen 443 default ssl;
        ssl_certificate tecssl/fullchain.pem;
        ssl_certificate_key tecssl/privkey.pem;

        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        server_name  www.zhangruibin.com zhangruibin.com;
        # other
        #error_page 497 https://$server_name$request_uri;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        #return 301 https://$server_name$request_uri;
        location / {
            # 反向代理到 82 端口
            proxy_pass http://127.0.0.1:82;
            add_header Strict-Transport-Security max-age=15768000;
            add_header Access-Control-Allow-Origin *;
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```

