# YuWriter关联GitHub

笔者从开始学习编程的时候就有断断续续记笔记的习惯。
从微软的OneNote，到有道云笔记，再到Boosnote,然后到现在的YuWriter，一代软件的更迭意味着一代技术的更迭，不可避免的是文件的管理混乱。

同时，随着看的东西越来越多，感觉自己知道的越来越少，几年前看过的一篇文章现在再看简直跟没看过一样。

所以才有了这篇文章，将自己整理的知识放到云端，这样想可以看到对某些内容的增删也能看到自己的程序猿历程。

上面说的那些纯属自己的思绪，各位看官下面进入正题。


如上面所说的，我现在是要将YuWrigter中的文档保存早Github上，那我这里先说一下什么是YuWrigter吧。
YuWrigter是一款功能较强大的Markdown编辑器，界面友好，支持很多自定义，总之这些介绍在百度上都能看到笔者在这里就不多说了，下载地址：

http://ivarptr.github.io/yu-writer.site/

可以按需下载。

那么接下来说怎么讲在YuWrigter上写的文章推送到Github上。


其实很简单：

YuWrigter支持添加本地文档库，所以我们的步骤是这样的：

1.新建本地文档库：YuWriterMdFile；
2.github新建仓库：YuWriterMdFile；
3.启动YuWriter，左下角有添加文档库，添加关联路径到本地文档库YuWriterMdFile；
4.新建文档：仓库基本介绍.md，并随意输入内容；
5.使用gitbash或者cmd窗口即可推送至GitHub仓库，具体的推送方法可以参考笔者以前写的一篇文章：https://blog.csdn.net/m0_37190495/article/details/79912212

笔者使用的版本是yu-writer-beta-0.5.3-windows-x64，不支持Git等version control插件，所以只能这样推送到GitHub上了，其他的有的markdown编辑器有的是支持git的，有兴趣的小伙伴可以自行探索，这里就不赘述了。

顺便说一句，笔者建立的这个仓库是用于存放自己写的以及网络上收集的优秀文章，持续更新，仓库地址：
https://github.com/zhangruibin/YuWriterMdFile

欢迎共同commit！！！