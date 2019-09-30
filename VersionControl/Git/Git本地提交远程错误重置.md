# Git本地提交远程错误重置


## 问题
今天笔者照例要把几个MD文件推送到GitHub库，但是经过add 和commit的再push到远程的时候出错了，看下面的错误信息：
```
$ git push origin master
fatal: HttpRequestException encountered.
   ▒▒▒▒▒▒▒▒ʱ▒▒▒▒
Username for 'https://github.com': itbin@foxmail.com
Counting objects: 22, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (18/18), done.
Writing objects: 100% (22/22), 1.83 MiB | 141.00 KiB/s, done.
Total 22 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 1 local object.
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
remote: error: Trace: 2b24782bfa4d3fee2c4cc96cd60b53d2
remote: error: See http://git.io/iEPt8g for more information.
remote: error: File Java/Cron/Cron.resource/commit.json is 257.43 MB; this exceeds GitHub's file size limit of 100.00 MB
To https://github.com/zhangruibin/YuWriterMdFile
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'https://github.com/zhangruibin/YuWriterMdFile'

```

可以看到，GitHub的单文件大小不能超过100M，我commit的文件有一个两百多M的，所以失败了，那咋办，本地已经commit了，那就只能将本地的commit回滚了。

## 解决办法
使用Git log 及git reset commitId来实现。


首先执行git log 命令，来查看log日志，日志是按照时间倒叙的，最后commit的在最上面，部分日志如下：
```
$  git log
commit eb13983f3b0c2cfba39ba2eeffb9001e3803086a (HEAD -> master)
Author: zhrb <itbin@foxmail.com>
Date:   Sun Sep 29 08:58:43 2019 +0800

    java 源码学习1

commit ad072b274daff241aa581285986f10734b4c8916
Author: zhrb <itbin@foxmail.com>
Date:   Sun Sep 29 08:42:54 2019 +0800

    java 源码学习1

commit d8191e0d0c3cbd1473b175804e2250d87036ba64 (origin/master)
Author: zhrb <itbin@foxmail.com>
Date:   Fri Sep 27 11:15:38 2019 +0800

    share

commit 695c8ebcc3a6c7089c257fc917e3d9fd19859fa6
Author: zhrb <itbin@foxmail.com>
Date:   Fri Sep 27 09:59:37 2019 +0800

    thread

```

可以看到远端仓库的commitId 为：d8191e0d0c3cbd1473b175804e2250d87036ba64

此时我们执行下面的命令：

$ git reset  d8191e0d0c3cbd1473b175804e2250d87036ba64

然后再执行git log可以看到，已经重置到跟远端仓库一致的版本了。

```
$ git log
commit d8191e0d0c3cbd1473b175804e2250d87036ba64 (HEAD -> master, origin/master)
Author: zhrb <itbin@foxmail.com>
Date:   Fri Sep 27 11:15:38 2019 +0800

    share

commit 695c8ebcc3a6c7089c257fc917e3d9fd19859fa6
Author: zhrb <itbin@foxmail.com>
Date:   Fri Sep 27 09:59:37 2019 +0800

    thread

```

本地commit 回退了版本之后，笔者处理掉那个两百多M的文件再重新commit，就跟之前的commit流程一样了，可以顺利的提交到Github了。




