# Github免输入账号密码提交代码Linux版


盆友们，看到上面的标题了吗 --》 Github免输入账号密码提交代码Linux版。
那我这里就真的是讲解怎么免输入认证提交GitHub吗？
不是的哟，来说下我这篇笔记是想做什么。

前因：
笔者时不时的登录自己的GitHub仓库，首页看 contributions in the last year模块时展示是这样的：

![](https://www.zhangruibin.com/upload/2019/10/sa6vna1q74g9apfcr683g6cri7.png)
感觉就是一片荒地，没有绿油油的苗，那肯定是满屏绿油油的苗好看啊，那咋整？笔者这也不是每天都有时间提交GitHub啊，毕竟自定义懒得一批。博文都断更两三天了，更重要的是，contributions 的绿色小块颜色程度也是有深度区别的，提交三次及以上，都是深绿色，和提交十次的颜色是一致的，那我想达到那种程度的深绿色就要每天contributions commit4次。
这点可以看下GitHub上说的统计帮助文档：点击contributions 那块下面的 ：Learn how we count contributions.
跳转连接：https://help.github.com/en/articles/why-are-my-contributions-not-showing-up-on-my-profile

阅读上面的连接可以看到统计规则，记住其中一点：
*   You are a collaborator on the repository or are a member of the organization that owns the repository.
这句话的大致意思是：提交者的身份必须是仓库协作者，或者拥有者。
这句话影响到之后我们设置Git全局变量。
-
好了，那现在简单的说下笔者想达到的目的：

每天有那么一个脚本，帮我自动commit几次代码到GitHub上，让我的contributions in the last year在我不主动提交代码的情况下也变的满屏绿油油的。

那我接下来就要开始自己的小动作了。

步骤：
1.准备一个Linux环境，安装好Git；
2.GitHub上建立一个仓库，并且在linux上建立仓库，然后使用https方式先关联仓库并commit成功；
3.写一个小脚本，运行脚本进行代码的commit；
4.设置Git全局变量及全局身份认证达到免输入账号密码提交代码的效果；
5.设置定时任务每天执行三次脚本commit代码到Github上；

那按照上面的步骤拆分的话，接下来的就简单了，一步一步走吧。


## 准备一个Linux环境，安装好Git

这步就不说了，很简单的，linux安装Git，这里就不赘述了。

## 建立仓库并关联

这个也不难，笔者之前的博客有写的，GitHub上也有介绍：
HTTPs的形式：
```
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/zhangruibin/testA.git
git push -u origin master
```

SSH形式：
```
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:zhangruibin/testA.git
git push -u origin master
```

## 脚本自动commit
说是写脚本，其实就是把Git的语句放到脚本里面可以一键执行：

```
echo 'first test'
#日期 
dd=`date +%Y%m%d%R`
cd /app/sel/blog/BlogMonitor
echo " $dd"
echo "Today is $dd ,it is still a nice day!">>BlogMonitorLog.log
git add *
git commit -m "$dd"
git push -u origin master

```

注意：
脚本中的echo "Today is $dd ,it is still a nice day!">>BlogMonitorLog.log，
>>意思是追加"Today is $dd ,it is still a nice day!"到BlogMonitorLog.log文件，如果是>那就是替换文件的内容。

此时运行脚本，我们还需要手动输入账号密码，这不对，我不想自己输入，那就设置全局身份认证吧。

## 免输入账号密码

GitHub免输入账号密码提交代码通常来说有两种方式，因为用命令行提交也是两种方式：HTTPS以及SSH。

那我们挨个来说：

### HTTPS形式免输入账号密码设置
#### 配置文件
linux下：
```
cd  ~/
touch .git-credentials
vim .git-credentials
```



编辑文件，增加内容为：
```
 https://{username}:{password}@github.com 

```


比如 https://account:password@github.com

#### 配置全局变量
git config --global credential.helper store
#### 验证
验证是否成功：

```
cat  ~/.gitconfig
```


如果成功会出现：
```
[credential]
helper = store
```

此时再执行脚本理论上就可以直接push成功了，而不需要账号密码。

### SSH方式

首先我们要确认仓库的提交方式是否是SSH的：
在本地仓库主目录下执行：git remote -v
笔者执行之后如下：
```
origin	git@github.com:zhangruibin/BlogMonitor.git (fetch)
origin	git@github.com:zhangruibin/BlogMonitor.git (push)

```

类似上面的就是SSH形式的，若是HTTPS的，则是含HTTPS。
如果仓库是HTTPS形式，则需要先手动切换为SSH形式。

切换方法：
1.登录GitHub仓库，切换为SSH形式。
2.修改本地仓库提交方式为SSH。
* 1、git remote -v  查看远程连接的方式；
* 2、git remote rm origin 删除原先HTTPS的连接方式；
* 3、GitHub仓库复制SSH的地址；
* 4、git remote add origin SSH地址，连接方式更改为SSH方式；
* 5、重新查看连接方式 git remote -v 会发现已经更改成了ssh的方式。

切换为SSH形式提交这时候我们要重新设置全局变量：
#### linux本地生成公钥

默认公钥是存储在用户目录下的.ssh目录中，如下：
/home/cp/.ssh # Linux

找不到的盆友可以使用whereis ssh来找。
本文件夹是隐藏文件夹，使用：ls -al可以查看详细信息。

找到之后进入到文件夹，此时里面只有一个文件：
known_hosts。
在此文件夹中执行命令：ssh-keygen -t rsa -C "youemail@example.com"
执行命令后会提示你输入保存文件的名字什么的，不用管，直接回车回车回车结束。
此时再看文件夹应该就多出来两个文件了：
多出来的两个文件名为：
id_rsa,id_rsa.pub
我们catid_rsa.pub的内容，并且复制，然后到我们的GitHub仓库，找到setting->SSH and GPGkeys，在这个选项里面点击新增一个key，然后把我们生成的pub key粘贴进去，命名并保存。

然后理论上也能达到免账号密码提交代码的效果了就。

此时到这一步，我们已经可以执行脚本免账号密码提价代码了，接下来的就剩下设置定时任务了。


一定要设置全局变量，设置用户名为仓库的协作者或者仓库的拥有者，否则本次commit不计数contribution统计：

```
git config --global user.name "nameVal"
git config --global user.email "eamil@qq.com"
```
查看git配置信息

git config --list
## 设置定时任务自动执行脚本

笔者买的VPS是centos7的，所以在选择定时任务的执行方式的时候笔者选择了其中一种方式Crontab。

cron服务是Linux的内置服务，但它不会开机自动启动。可以用以下命令启动和停止服务：

systemctl start crond

systemctl stop crond

systemctl restart crond

systemctl reload crond

以上1-4行分别为启动、停止、重启服务和重新加载配置。

要把cron设为在开机的时候自动启动，在 /etc/rc.d/rc.local 脚本中加入 `/sbin/service crond start` 即可

查看当前用户的crontab，输入 crontab -l；

编辑crontab，输入 crontab -e；

删除crontab，输入 crontab -r

添加任务

  crontab -e
  0 */1 * * * command
  0 */2 * * * command

查询任务是否加了：

  crontab -l -u root #查看root用户
  0 */1 * * * command
  0 */2 * * * command


比如笔者，就是添加了如下：

```
0,30 8-9 * * * ./app/sel/blog/autoCommitToGithub.sh
```

也就是每天八点到九点之间，每小时执行一次脚本提交代码到GitHub上。

到这里就完事了，再也不用看着一片荒地没有小绿点啦~~~~





