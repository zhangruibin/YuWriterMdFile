 <blockquote>
转载请标明出处^_^<br> 原文首发于：<a href="https://www.zhangruibin.com" rel="nofollow" target="_blank">https://www.zhangruibin.com</a><br> 本文出自于：<a href="https://www.zhangruibin.com" rel="nofollow" target="_blank">RebornChang的博客</a>
</blockquote>

# MySQL服务中断后重启失败及解决办法(个例)
 
 # Starting MySQL. ERROR! The server quit without updating PID file (/usr/local/mysql/mysql-5.6.44-linux-glibc2.12-x86_64/data/zhrb.localdomain.pid).
 早上上班的时候习惯性刷下自己的博客看看出故障没，有没有新的留言，刷了之后发现出故障了，那么按照以往的经验是数据库死掉了，那直接重启数据库吧。
 因为MySQL服务加入了service，所以直接service mysql start 启动吧，意外的是，启动失败了，启动结果：
 Starting MySQL. ERROR! The server quit without updating PID file (/usr/local/mysql/mysql-5.6.44-linux-glibc2.12-x86_64/data/zhrb.localdomain.pid).
# 排查失败原因
根据提示，系统启动的时候找不到.pid结尾的文件，去文件夹下面ll的确没这个文件，但是为什么没这个文件，找到原因才能解决问题，所以还是看日志吧，博主的安装目录为/usr/local/mysql/mysql-5.6.44-linux-glibc2.12-x86_64/data/，错误日志指定在该目录下，查看zhrb.localdomain.err日志，截取相关日期的错误日志记录如下：
 ```
 2019-08-11 20:29:06 11832 [Note] Plugin 'FEDERATED' is disabled.
2019-08-11 20:29:06 11832 [Note] InnoDB: Using atomics to ref count buffer pool pages
2019-08-11 20:29:06 11832 [Note] InnoDB: The InnoDB memory heap is disabled
2019-08-11 20:29:06 11832 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2019-08-11 20:29:06 11832 [Note] InnoDB: Memory barrier is not used
2019-08-11 20:29:06 11832 [Note] InnoDB: Compressed tables use zlib 1.2.11
2019-08-11 20:29:06 11832 [Note] InnoDB: Using Linux native AIO
2019-08-11 20:29:06 11832 [Note] InnoDB: Using CPU crc32 instructions
2019-08-11 20:29:06 11832 [Note] InnoDB: Initializing buffer pool, size = 128.0M
InnoDB: mmap(137363456 bytes) failed; errno 12
2019-08-11 20:29:06 11832 [ERROR] InnoDB: Cannot allocate memory for the buffer pool
2019-08-11 20:29:06 11832 [ERROR] Plugin 'InnoDB' init function returned error.
2019-08-11 20:29:06 11832 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2019-08-11 20:29:06 11832 [ERROR] Unknown/unsupported storage engine: InnoDB
2019-08-11 20:29:06 11832 [ERROR] Aborting

2019-08-11 20:29:06 11832 [Note] Binlog end
2019-08-11 20:29:06 11832 [Note] Shutting down plugin 'partition'
2019-08-11 20:29:06 11832 [Note] Shutting down plugin 'ARCHIVE'
2019-08-11 20:29:06 11832 [Note] Shutting down plugin 'BLACKHOLE'
2019-08-11 20:29:06 11832 [Note] Shutting down plugin 'INNODB_SYS_DATAFILES'

 ```
 
 通过上面的日志我们可以看到一句重要的话：InnoDB: Initializing buffer pool, size = 128.0M
InnoDB: mmap(137363456 bytes) failed; errno 12

 这句话说的是InnoDB数据库引擎的初始工作区缓存大小为128M,系统可分配的mmap内存大小为137363456 bytes，换算成M是131M。
 理论上来看，131M>128M，不应该存在分配内存失败啊，但是就是失败了，那是不是跟系统的内存分配机制有关？
 顺便提一句，博主买的低端主机配置为：2Core，1G RAM,满服务下内存使用率如下图所示：
 ![](https://www.zhangruibin.com/upload/2019/08/439elaicleh62o3imnggo620kd.png)
 # 分析原因可能性
 这时我们看下系统可用内存free -m：
 ```
 [root@zhrb mysql-5.6.44-linux-glibc2.12-x86_64]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1007         414         469           2         123         442
Swap:           259         233          26
```
理论上看下是此时是有足够的内存去分配的（注意：此主机上运行了两个博客程序，每个分配的内存为300M，也就是说，剩下的内存虽然多，但是那是被别的程序打上标记了，一旦负载量大，别的程序占用了更多的内存就会跟mysql竞争，就容易导致分配不了足够内存进而分配失败）
但是依然启动失败，那是不是系统的内存分配策略不是有足够内存就全量分配呢？

来看一下：
```
[root@zhrb mysql-5.6.44-linux-glibc2.12-x86_64]# cat /proc/sys/vm/overcommit_memory 
0
```
那这个overcommit_memory参数是什么？
 默认值为：0
 从内核文档里得知，该参数有三个值，分别是：

 0：当用户空间请求更多的的内存时，内核尝试估算出剩余可用的内存；

 1：当设这个参数值为1时，内核允许超量使用内存直到用完为止，主要用于科学计算；

 2：当设这个参数值为2时，内核会使用一个决不过量使用内存的算法，即系统整个内存地址空间不能超过swap+50%的RAM值，50%参数的设定是在overcommit_ratio中设定。

也就是说主机内存分配策略也能保证有足够内存的话全量分配，那就不是分配策略的问题。
 # 尝试可行性解决办法
依照上面所述总结结果：
当发生内存竞争时，innoDB的内存如果获取不足就会导致MySQL服务中断。

解决办法：

减小innoDB内存。

解决方式：

修改/etc/my.cnf

文件部分内容如下图所示：

![](https://www.zhangruibin.com/upload/2019/08/453auuev8qhtqpmugfog798jcn.png)

放开注释并修改大小为50M，保存退出。

重启MySQL服务：
```
[root@zhrb mysql-5.6.44-linux-glibc2.12-x86_64]# service mysql start
Starting MySQL.. SUCCESS! 
```

启动成功。

本次故障排查及解决过程简单截图如下：
![](https://www.zhangruibin.com/upload/2019/08/5dedefjr58hanpp78u2eh8okds.png)




 
 
 
 