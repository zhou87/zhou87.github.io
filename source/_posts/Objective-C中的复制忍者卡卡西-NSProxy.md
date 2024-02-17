---
title: Objective-C中的复制忍者卡卡西---NSProxy
categories: iOS
date: 2017-12-28 11:24:12
comments: true
tags:
    - NSProxy
---

是不是曾经在别人面前夸下海口：`Objective-C`中所有的类的基类都是`NSObject`;我之前也一直也这么以为的，但是认真看了下官方文档：


![](/图片测试/A14B3BD8-1FB4-4F2B-BB4E-B259CA3B4DD7.png)


啪啪，好响，好疼~（看来英文水平也很重要啊......）
不卖关子了，我们今天要讨论的就是`NSProxy`。它是跟`NSObject`属于同一级别的类，是个抽象类，只是实现了`<NSObject>`的协议;
<!-- more -->
![](/图片测试/A9761DE2-782E-4900-A364-C2C008FEDE96.png)

按照官方的定义：`NSProxy`是一个为对象定义接口的抽象父类，并且为其它对象或者一些不存在的对象扮演了替身角色。通常，给proxy的消息被转发给实际对象或者导致proxy加载（转化它为）实际对象。`NSProxy`的子类能被用来实现透明的分布式消息(例如:NSDistantObject)或者延缓要花费昂贵代价创建的对象的实现。

下面我们看看`NSProxy`是怎么复制别的类（几乎所有类）的：

我们先创建一个类继承自`NSProxy`:

```java
#import "KakashiProxy.h"

@interface KakashiProxy ()

@property(nonatomic,strong)NSObject *objc;

@end

@implementation KakashiProxy

- (void)changeObj:(NSObject *)obj {
    
    self.objc = obj;
    
}

//方法签名
- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel {
    
    NSMethodSignature *signature = nil;
    if ([self.objc methodSignatureForSelector:sel]) {
        signature = [self.objc methodSignatureForSelector:sel];
    }else {
        signature = [super methodSignatureForSelector:sel];
    }
    
    return signature;
}

//调用方法实现
- (void)forwardInvocation:(NSInvocation *)invocation {
    
    if ([self.objc respondsToSelector:invocation.selector]) {
        NSString *selName = NSStringFromSelector(invocation.selector);
        NSLog(@"selector name : %@",selName);
        //这里我们可以做一些逻辑处理，比如埋点统计之类的
        [invocation invokeWithTarget:self.objc];
    }else {
        [super forwardInvocation:invocation];
    }
    
}

@end
```
可以看到我们重写了两个方法：`methodSignatureForSelector:`和`forwardInvocation:(NSInvocation *)invocation`,前者是实现方法签名的，我们可以在这个方法中，直接将我们复制的对象进行方法签名，然后生成`NSInvocation`。接着方法`forwardInvocation`会被调用，在这个方法中，我们直接将`invacation`的`target`设置为我们复制的对象。在这个方法中我们可以做一点事情，比如根据类名做一些逻辑，也可以做一些数据埋点之类的。
然后我们看看怎么使用这个复制忍者的：
我们随便定义两个类，并为每个类设置了一个方法：

`Parker-Dog.h`

```Java
    #import "Parker-Dog.h"
    
    @implementation Parker_Dog
    
    - (void)bite {
        NSLog(@"卡卡西召唤通灵兽Paker并咬住了再不斩!");
    }
    
    @end
```

`Wood.h`

```java
    #import "Wood.h"

    @implementation Wood
    
    - (void)boom{
    
        NSLog(@"变!木头!...");
        
    }
```
调用：

```java
    //初始化一个木头
    Wood *wood = [[Wood alloc] init];
    //初始化一个帕克
    Parker_Dog  *paker = [[Parker_Dog alloc] init];
    //初始化一个卡卡西
    KakashiProxy *proxy = [KakashiProxy alloc];
    
    //变木头
    [proxy changeObj:wood];
    //调用木头的方法
    [proxy performSelector:@selector(boom) withObject:nil];
    
    //变帕克
    [proxy changeObj:paker];
    //调用帕克的方法
    [proxy performSelector:@selector(bite) withObject:nil];
```
控制台输出:
![](/图片测试/4CBD3398-72E0-4E8B-AB41-045CFC4468FA.png)

这样，我们通过`proxy`这个类实现了复制`wood`和`paker`，可以分别调用各个类的方法了。

### 总结
其实也不能叫复制吧，大家也可以看到，这个实现其实就是利用了`runtime`的消息转发机制。这篇文章就算是对`NSProxy`的一个简单认识吧，还有些`NSProxy`的高级用法可能笔者还没有学习到，还要继续努力，有时间需要深入研究下这个特殊的类。

>参看文章


* [NSProxy](https://developer.apple.com/documentation/foundation/nsproxy?language=objc)
* [用 NSProxy 实现面向切面编程](https://www.jianshu.com/p/a7187e014c03)


