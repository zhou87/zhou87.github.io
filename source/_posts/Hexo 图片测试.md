---
title: Hexo 图片测试
date: 2017-04-21 10:24:54
categories: 
tags:
 - hexo博客搭建
keywords:
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=415792881&auto=0&height=66"></iframe>
(莫名其妙就想设置背景音乐...)
hexo插入图片好像是个问题，使用绝对路径相对路径都行不通（貌似）。
遂百度了下得知可以用hexo的 asset功能，具体实现如下：
 1.在hexo根目录下的_config.yml文件中将 post_asset_folder参数设置为true:
```
post_asset_folder: true
```
2.然后安装插件CodeFalling/hexo-asset-image：
hexo目录下执行:
```
npm install https://github.com/CodeFalling/hexo-asset-image --save
```

<!-- more -->

3.安装完成后用hexo新建文章的时候会在_posts目录下多出一个和文章名字一样的文件夹，图片就可以放在这个文件夹里面:

![WechatIMG2](/%E5%9B%BE%E7%89%87%E6%B5%8B%E8%AF%95/WechatIMG2.jpeg)
4.
```
![](图片测试/aNou.jpg)
```
这样就可以插入图片了。




但是
=======
这种方法看似简单，然后当我安装好插件之后执行 hexo g的时候，终端给我抛出一段错误:
```
FATAL Cannot read property 'replace' of undefined
TypeError: Cannot read property 'replace' of undefined
    at Object.<anonymous> (/Users/rongfeng/Desktop/Blog/hexo/node_modules/hexo-asset-image/index.js:31:38)
    at initialize.exports.each (/Users/rongfeng/Desktop/Blog/hexo/node_modules/hexo-asset-image/node_modules/cheerio/lib/api/traversing.js:293:24)
    at Hexo.<anonymous> (/Users/rongfeng/Desktop/Blog/hexo/node_modules/hexo-asset-image/index.js:29:16)
    at Hexo.tryCatcher (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/util.js:16:23)
    at Hexo.<anonymous> (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/method.js:15:34)
    at /Users/rongfeng/Desktop/Blog/hexo/node_modules/hexo/lib/extend/filter.js:68:35
    at tryCatcher (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/util.js:16:23)
    at Object.gotValue (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/reduce.js:155:18)
    at Object.gotAccum (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/reduce.js:144:25)
    at Object.tryCatcher (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/util.js:16:23)
    at Promise._settlePromiseFromHandler (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/promise.js:512:31)
    at Promise._settlePromise (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/promise.js:569:18)
    at Promise._settlePromise0 (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/promise.js:614:10)
    at Promise._settlePromises (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/promise.js:693:18)
    at Async._drainQueue (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/async.js:133:16)
    at Async._drainQueues (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/async.js:143:10)
    at Immediate.Async.drainQueues (/Users/rongfeng/Desktop/Blog/hexo/node_modules/bluebird/js/release/async.js:17:14)
    at runCallback (timers.js:672:20)
    at tryOnImmediate (timers.js:645:5)
    at processImmediate [as _immediateCallback] (timers.js:617:5)
```
然后
====
 就没有然后了，也查了点资料，然并卵。只好另寻他路。。。

然而
====
 不久，便发现了一个不错的工具-----MWeb;
 插入图片拖进文件就可以，它自动会填充好代码，同时他能支持图形界面的查看，所见即所得，这正是我最近想找的一个Markdown编辑器。
  需要注意的一点是，在导入soure文件的时候指定一个图片的存放路径即可：
 ![WechatIMG4](/%E5%9B%BE%E7%89%87%E6%B5%8B%E8%AF%95/WechatIMG4.jpeg)

![WechatIMG5](/%E5%9B%BE%E7%89%87%E6%B5%8B%E8%AF%95/WechatIMG5.jpeg)


 况且，MWeb的图像界面看上去也挺友好的，上手快，确实是个不错的Markdown编辑器。
![WechatIMG3](/%E5%9B%BE%E7%89%87%E6%B5%8B%E8%AF%95/WechatIMG3.jpeg)


总结
=====
记录下折腾的过程，大家有什么好的方法给我给我介绍介绍，😆。


