---
title: 折腾Hexo过程中的一些坑
comments: true
date: 2017-04-17 19:08:48
categories:
tags: 
     - hexo博客搭建
keywords:

---

  今天一时兴起也花了点时间折腾了下Hexo
============
  对于刚有想写点Blog的我来说，那种像[王巍](https://onevcat.com/)比较装X的博客还是比较有吸引力的。那时候心里就想，大佬就是大佬，人家的博客都与众不同。
也是一个偶然的机会，也怪我平时会在[简书]()这种平台瞎逛，看到了一片关于用Hexo搭建博客的文章(文章地址:[5分钟 搭建免费个人博客](http://www.jianshu.com/p/4eaddcbe4d12))。文章标题很吸引人，我就是这么被忽悠了然后点进去看了下（实话说，简书的标题党还是有的）。

<!-- more -->

  就这样，跟着作者的思路和步奏一步一步在Mac走下去，折腾折腾，之所以叫折腾，肯定会在过程中遇到点问题是吧。下面问题就开始了：

  >安装hexo的时候提示：node已经安装好了，只是没有“linked”

   解决方法：可能是好久之前安装的homebrew，所以我更新了下homebrew,再按照这个走一遍，好像可以了


  >第二个问题就是安装hexo的时候提示找不到文件的错误:像这种：
  ```
   { Error: Cannot find module './build/Release/DTraceProviderBindings'
    at Function.Module._resolveFilename (module.js:470:15)
    at Function.Module._load (module.js:418:25)
    at Module.require (module.js:498:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/Users/rongfeng/Desktop/博客/node_modules/dtrace-provider/dtrace-provider.js:17:23)
    at Module._compile (module.js:571:32)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.require (module.js:498:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/Users/rongfeng/Desktop/博客/node_modules/bunyan/lib/bunyan.js:79:18)
    at Module._compile (module.js:571:32)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.require (module.js:498:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/Users/rongfeng/Desktop/博客/node_modules/hexo-log/lib/log.js:3:14)
    at Module._compile (module.js:571:32) code: 'MODULE_NOT_FOUND' }
{ Error: Cannot find module './build/default/DTraceProviderBindings'
    at Function.Module._resolveFilename (module.js:470:15)
    at Function.Module._load (module.js:418:25)
    at Module.require (module.js:498:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/Users/rongfeng/Desktop/博客/node_modules/dtrace-provider/dtrace-provider.js:17:23)
    at Module._compile (module.js:571:32)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.require (module.js:498:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/Users/rongfeng/Desktop/博客/node_modules/bunyan/lib/bunyan.js:79:18)
    at Module._compile (module.js:571:32)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.require (module.js:498:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/Users/rongfeng/Desktop/博客/node_modules/hexo-log/lib/log.js:3:14)
    at Module._compile (module.js:571:32) code: 'MODULE_NOT_FOUND' }
{ Error: Cannot find module './build/Debug/DTraceProviderBindings'
    at Function.Module._resolveFilename (module.js:470:15)
    at Function.Module._load (module.js:418:25)
    at Module.require (module.js:498:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/Users/rongfeng/Desktop/博客/node_modules/dtrace-provider/dtrace-provider.js:17:23)
    at Module._compile (module.js:571:32)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.require (module.js:498:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/Users/rongfeng/Desktop/博客/node_modules/bunyan/lib/bunyan.js:79:18)
    at Module._compile (module.js:571:32)
    at Object.Module._extensions..js (module.js:580:10)
    at Module.load (module.js:488:32)
    at tryModuleLoad (module.js:447:12)
    at Function.Module._load (module.js:439:3)
    at Module.require (module.js:498:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/Users/rongfeng/Desktop/博客/node_modules/hexo-log/lib/log.js:3:14)
    at Module._compile (module.js:571:32) code: 'MODULE_NOT_FOUND' }
  ```
  **解决方法就是：**

  ```
  $ npm install hexo --no-optional

  ```
 用上面的方法从新安装一遍hexo就可以了


  >最后个坑:本地“localhost:4000”是可以的，没问题，但是部署到GitHub上就是一直显示“test github page”

   这个是主要问题,期间我又去查找了别的资料，比如这位博主的[Mac搭建Hexo博客及NexT主题配置优化](http://www.jianshu.com/p/fb0b0258362f)。他们两个的内容应该说是差不多的。最后给我很大帮组的一片博客还属这位大哥:[手把手教你使用Hexo + Github Pages搭建个人独立博客](http://blog.csdn.net/lzrreach/article/details/52863798)

   这里面提到关键的地方是 **hexo deploy部署 会偶尔莫名其妙的出问题**  **使用git命令行部署更有效**
  
```
clone github repo
$ cd d:/hexo/blog

$ git clone https://github.com/jiji262/jiji262.github.io.git .deploy/jiji262.github.io
将我们之前创建的repo克隆到本地，新建一个目录叫做.deploy用于存放克隆的代码。

创建一个deploy脚本文件
hexo generate
cp -R public/* .deploy/jiji262.github.io
cd .deploy/jiji262.github.io
git add .
git commit -m “update”
git push origin master
简单解释一下，hexo generate生成public文件夹下的新内容，然后将其拷贝至jiji262.github.io的git目录下，然后使用git commit命令提交代码到jiji262.github.io这个repo的master branch上。

```
 确实是这样，每次用
 ```
 hexo d

 ```
都报这样的错误:
```
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
[master 8f36dd3] Site updated: 2017-04-17 17:35:41
 6 files changed, 18 insertions(+), 13 deletions(-)
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

    at ChildProcess.<anonymous> (/Users/rongfeng/Desktop/Blog/hexo/node_modules/hexo-util/lib/spawn.js:37:17)
    at emitTwo (events.js:106:13)
    at ChildProcess.emit (events.js:194:7)
    at maybeClose (internal/child_process.js:899:16)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:226:5)
FATAL Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

Error: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

    at ChildProcess.<anonymous> (/Users/rongfeng/Desktop/Blog/hexo/node_modules/hexo-util/lib/spawn.js:37:17)
    at emitTwo (events.js:106:13)
    at ChildProcess.emit (events.js:194:7)
    at maybeClose (internal/child_process.js:899:16)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:226:5)
```
然而 按上面那种方法可以立马解决这个问题，也让我初次尝到了把博客部署到github上的快感，感谢这位大佬！

 总结
 =======   

  笔记也就记到这里，主要是把在折腾过程中遇到的一些花了点时间去找解决方法的点记录下，其他的步奏前面提到的博课都写得比较详细。还有就是后面了解了下markdown的语法和用法，按照这个网站上的知识看一遍应该有所帮助：[MarkDown](http://xianbai.me/learn-md/article/syntax/headers.html)


   



