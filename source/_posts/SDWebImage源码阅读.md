---
title: SDWebImage源码阅读(上)
comments: true
categories: iOS
date: 2018-01-11 10:54:05
tags:
    - 源码
    - SDWebImage
---

### 前言
![SDWebImage](https://raw.githubusercontent.com/rs/SDWebImage/master/SDWebImage_logo.png)
***SDWebImage*** 想必对于每一个iOS开发者来说，都不会陌生。每当我们的项目中需要给UIImageView设置网络图片的时候，我们基本上第一个想到就是这个强大的第三方库。我们可以手动导入静态库，也可以直接用CocoaPod管理，只要简单地输入`pod 'SDWebImage'`即可导入，大家可以随意选择。
虽然在项目中经常用到这个库，但是有时候我们可能只是知道用，却并没有去了解下这个强大库实现原理，或者是写这个库的大佬代码思路、风格和各种技巧。这篇文章就是从`SDWebImage`一行行源码来分析大佬的神级操作。
### SDWebImage流程示意图
![SDWebImage流程示意图](http://ooqxuxrxe.bkt.clouddn.com/SDWebImageSequenceDiagram.png)
<!-- more -->
### 从sd_setImageWithURL开始

在我们将`SDWebImage`导入到项目中的时候，点开`core`文件夹可能会看到一大串文件，刚开始肯定觉得很头疼，这么多文件，这么多文件，我要看到什么时候去，畏难心涌上来了，很多同学就在这里倒下，把`Xcode`一关，然后干别的事去了...。其实这样很正常，不过我们可以整理下，其实`SDWebImage`真正核心的也就几个文件，我们可以把他们单独拿出来，比如这样
![SDWebImage文件整理](http://ooqxuxrxe.bkt.clouddn.com/SDWebImage%E6%96%87%E4%BB%B6%E6%95%B4%E7%90%86.jpg)

这样可能会看得舒服点。把文件整理好了，那我们就开始看下每个文件有什么用，干什么的了。我们先从最常用的方法开始：`sd_setImageWithURL`

我们随便布局一个UIImageView，然后设置图片：

```
UIImageView *img = [[UIImageView alloc] init];
img.frame = CGRectMake(0, 0, 100, 100);
img.center = self.view.center;
img.backgroundColor = [UIColor orangeColor];
[self.view addSubview:img];
//加载图片
[img sd_setImageWithURL:[NSURL URLWithString:@"https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1516084683&di=aeeadb9d3ae89483d06792970a46f829&imgtype=jpg&er=1&src=http%3A%2F%2Fp3.qhimg.com%2Ft010f0383e8e5f9bdbb.png"] placeholderImage:nil];
```
我们可以点进这个方法看看是怎么实现的：

```
- (void)sd_setImageWithURL:(nullable NSURL *)url placeholderImage:(nullable UIImage *)placeholder {
    [self sd_setImageWithURL:url placeholderImage:placeholder options:0 progress:nil completed:nil];
}
```
然后一路点下去，你会发现，它们最终都会调到UIView+WebCache的`sd_internalSetImageWithURL`方法里面来执行,我们看看这里面的代码：

```
- (void)sd_internalSetImageWithURL:(nullable NSURL *)url
                  placeholderImage:(nullable UIImage *)placeholder
                           options:(SDWebImageOptions)options
                      operationKey:(nullable NSString *)operationKey
                     setImageBlock:(nullable SDSetImageBlock)setImageBlock
                          progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                         completed:(nullable SDExternalCompletionBlock)completedBlock
                           context:(nullable NSDictionary *)context {
    //获取到绑定的key
    NSString *validOperationKey = operationKey ?: NSStringFromClass([self class]);
    //取消当前进行的操作组合
    [self sd_cancelImageLoadOperationWithKey:validOperationKey];
    //用imageURLKey做key,将url作为属性绑定到UIView上
    objc_setAssociatedObject(self, &imageURLKey, url, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    
    //“&”在这里是个位运算，if括号里面的运算表示options的值不是“延迟占位图展示”的话的话就成立。
    if (!(options & SDWebImageDelayPlaceholder)) {
        //宏定义，回到主线程中设置图片(设置占位图)
        dispatch_main_async_safe(^{
            [self sd_setImage:placeholder imageData:nil basedOnClassOrViaCustomSetImageBlock:setImageBlock];
        });
    }
    
    if (url) {
        // check if activityView is enabled or not
        //简单翻译就是 设置了activityView（正在加载指示器）为显示的话，那么就调用显示activityView的方法...
        if ([self sd_showActivityIndicatorView]) {
            //去创建菊花（activityView）然后转起来...
            [self sd_addActivityIndicator];
        }
        
        __weak __typeof(self)wself = self;
        //下载图片的方法
        id <SDWebImageOperation> operation = [SDWebImageManager.sharedManager loadImageWithURL:url options:options progress:progressBlock completed:^(UIImage *image, NSData *data, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
            
            __strong __typeof (wself) sself = wself;
            //先把菊花停掉...
            [sself sd_removeActivityIndicator];
            //如果 self被提前释放了，直接返回
            if (!sself) { return; }
            //创建个bool变量 shouldCallCompletedBlock 只要图片下载完成或者options设置为不需要自动设置图片,它的值也为真。
            BOOL shouldCallCompletedBlock = finished || (options & SDWebImageAvoidAutoSetImage);
            //同样 如果有图片但是options值为SDWebImageAvoidAutoSetImage，或者没有返回图片且options值为SDWebImageDelayPlaceholder 值为真。
            BOOL shouldNotSetImage = ((image && (options & SDWebImageAvoidAutoSetImage)) ||
                                      (!image && !(options & SDWebImageDelayPlaceholder)));
            
            //一个block（执行完成回调）
            SDWebImageNoParamsBlock callCompletedBlockClojure = ^{
                
                //如果 self被提前释放了，直接返回
                if (!sself) { return; }
                //需要直接设置图片的话，直接调用setNeedsLayout
                if (!shouldNotSetImage) {
                    [sself sd_setNeedsLayout];
                }
                //将图片用block回调回去。
                if (completedBlock && shouldCallCompletedBlock) {
                    completedBlock(image, error, cacheType, url);
                }
            };
            
            // case 1a: we got an image, but the SDWebImageAvoidAutoSetImage flag is set
            // OR
            // case 1b: we got no image and the SDWebImageDelayPlaceholder is not set
            
            if (shouldNotSetImage) {
                //主线程中执行 callCompletedBlockClojure block
                dispatch_main_async_safe(callCompletedBlockClojure);
                return;
            }
            
            UIImage *targetImage = nil;
            NSData *targetData = nil;
            //如果我们获取到了图片，而且没有设置“禁止自动设置图片”，则将图片赋值给targetImage,将图片数据赋值给targetData.
            if (image) {
                // case 2a: we got an image and the SDWebImageAvoidAutoSetImage is not set
                targetImage = image;
                targetData = data;
            }
            //如果我们没有获取到图片，而且我们设置了“延迟设置占位图”,则将占位图赋值给targetImage,targetData设置为nil.
            else if (options & SDWebImageDelayPlaceholder) {
                // case 2b: we got no image and the SDWebImageDelayPlaceholder flag is set
                targetImage = placeholder;
                targetData = nil;
            }
            
            BOOL shouldUseGlobalQueue = NO;
            //判断是否设置图片需要在并发对列中执行
            //context是一个字典，可以设置key为“SDWebImageInternalSetImageInGlobalQueueKey”的值
            if (context && [context valueForKey:SDWebImageInternalSetImageInGlobalQueueKey]) {
                shouldUseGlobalQueue = [[context valueForKey:SDWebImageInternalSetImageInGlobalQueueKey] boolValue];
            }
            //三目运算，如果需要在全局并发队列执行则获取获取全局并发队列赋值给targetQueue,否则赋值主线程。
            dispatch_queue_t targetQueue = shouldUseGlobalQueue ? dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0) : dispatch_get_main_queue();
            
            dispatch_async(targetQueue, ^{
                //在目标队列中设置图片
                [sself sd_setImage:targetImage imageData:targetData basedOnClassOrViaCustomSetImageBlock:setImageBlock];
                //主线程回调
                dispatch_main_async_safe(callCompletedBlockClojure);
            });
        }];
        //为UIView绑定新的操作，因为之前把UIView上的操作取消了。
        [self sd_setImageLoadOperation:operation forKey:validOperationKey];
        
    } else {
        //如果url为空的话，
        dispatch_main_async_safe(^{
            //停止转菊花
            [self sd_removeActivityIndicator];
            if (completedBlock) {
                NSError *error = [NSError errorWithDomain:SDWebImageErrorDomain code:-1 userInfo:@{NSLocalizedDescriptionKey : @"Trying to load a nil url"}];
                //将错误信息回调回去
                completedBlock(nil, error, SDImageCacheTypeNone, url);
            }
        });
    }
}
```
这个方法里面代码有点多，刚开始一般过一遍都比较模棱两可。不过跟着注释应该能了解个大概的流程了。其实，总的来说这个方法差不多做了这三件事：
>1.根据`key`取消当前的操作组合，
>2.将`url`作为属性绑定到`UIView`上
>3.`SDWebImageManager`根据url下载图片

我们一个一个来看看：
##### 1.取消当前操作
```
//根据key取消当前进行的操作组合
[self sd_cancelImageLoadOperationWithKey:validOperationKey];
```
这个很有必要，举个栗子：当`tableView`的`cell`包含了`UIImageView`被重用的时候，首先会执行这个方法，保证了下载和缓存操作组合被取消了。
当我们点进方法去看可以看到它是在`UIView+WebCacheOperation`里实现的：

```
- (void)sd_cancelImageLoadOperationWithKey:(nullable NSString *)key {
    // Cancel in progress downloader from queue
    //遍历UIView上的所有正在下载的队列，并取消队列
    SDOperationsDictionary *operationDictionary = [self operationDictionary];
    id operations = operationDictionary[key];
    if (operations) {
        if ([operations isKindOfClass:[NSArray class]]) {
            for (id <SDWebImageOperation> operation in operations) {
                if (operation) {
                    [operation cancel];
                }
            }
        } else if ([operations conformsToProtocol:@protocol(SDWebImageOperation)]){
            [(id<SDWebImageOperation>) operations cancel];
        }
        //取消队列之后，移除字典中key对应的Value，也就是所有的operations
        [operationDictionary removeObjectForKey:key];
    }
}
```
* 在这个方法里面有一个`SDOperationsDictionary`类型的`operationDictionary`字典，其实`SDOperationsDictionary`是`NSMutableDictionary`的一个类型别名，它这样定义的：

```
typedef NSMutableDictionary<NSString *, id> SDOperationsDictionary;
```
这里`[self operationDictionary]`返回了一个字典赋给了`operationDictionary`

* 看看`operationDictionary`方法的实现:

```
- (SDOperationsDictionary *)operationDictionary {
    
    //SDOperationsDictionary是NSMutableDictionary的一个类型别名，跟之前版本有点不一样
    //这里用到了runtime的类型绑定,给UIView绑定了一个SDOperationsDictionary的operations
    //先通过key来获取UIView的operations属性值
    SDOperationsDictionary *operations = objc_getAssociatedObject(self, &loadOperationKey);
    //如果能获取到值，直接返回
    if (operations) {
        return operations;
    }
    //如果没有值，则初始化一个空的字典绑定给UIView,然后返回
    operations = [NSMutableDictionary dictionary];
    objc_setAssociatedObject(self, &loadOperationKey, operations, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    return operations;
}
```
注释还算清楚，用`runtime`给`UIView`绑定了一个`operation`字典，字典的`key`是针对针对不同类型的试图和不同类型的操作设定的字符串，字典的`value`是具体的操作（`operation`）。
##### 2.将`url`作为属性绑定到`UIView`上
```
//用imageURLKey做key,将url作为属性绑定到UIView上
objc_setAssociatedObject(self, &imageURLKey, url, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
```
这里也是利用了下`runtime`的类型绑定，用`imageURLKey`做为`key`,将`url`绑定到`UIView`上。这样，我们可以通过当前的`UIView`直接获取图片的`url`。
##### 3.根据`url`下载图片
```
//下载图片的方法
id <SDWebImageOperation> operation = [SDWebImageManager.sharedManager loadImageWithURL:url options:options progress:progressBlock completed:^(UIImage *image, NSData *data, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
}
```
我们可以看到，它是由一个`SDWebImageManager`单例来调用`loadImageWithURL`方法，最后会返回一个`SDWebImageOperation`类类型的`operation`。
>***1.我们点进去看看这个`SDWebImageManager`是怎么初始化的：***

```
//单例
+ (nonnull instancetype)sharedManager {
    static dispatch_once_t once;
    static id instance;
    dispatch_once(&once, ^{
        instance = [self new];
    });
    return instance;
}

//初始化，创建一个SDImageCache单例和SDWebImageDownloader单例,
- (nonnull instancetype)init {
    SDImageCache *cache = [SDImageCache sharedImageCache];
    SDWebImageDownloader *downloader = [SDWebImageDownloader sharedDownloader];
    return [self initWithCache:cache downloader:downloader];
}

- (nonnull instancetype)initWithCache:(nonnull SDImageCache *)cache downloader:(nonnull SDWebImageDownloader *)downloader {
    if ((self = [super init])) {
        //赋值SDImageCache和SDWebImageDownloader
        _imageCache = cache;
        _imageDownloader = downloader;
        //新建一个NSMutableSet来存储下载失败的url.
        _failedURLs = [NSMutableSet new];
        //新建一个NSMutableArray存储下载中的Operation
        _runningOperations = [NSMutableArray new];
    }
    return self;
}
```
在初始化中，我们还初始化了四个了变量：

* `SDImageCache`类型的`_imageCache`，实现缓存管理的变量；
* `SDWebImageDownloader`类型的`_imageDownloader`，实现图片下载的变量；
* `NSMutableSet`类型的`_failedURLs`，存那些失败的url的变量；
* `NSMutableArray`类型的`_runningOperations`，存放正在执行的任务的比变量。

这里要啰嗦下的就是为什么`_failedURLs`要用`NSMutableSet`类型而不用`NSMutableArray`类型。因为我们在搜索列表中元素的时候，NSSet要比NSArray更高效，NSSet里面的实现用到hash（哈希）算法，存和取都可以快速的定位。而在NSSArray中，要找其中某个元素，基本上都是要把所有元素遍历一遍，很明显的效率就低了很多。还有就是NSSet里面不会有重复的元素，同一个下载失败的url也只会存在一个。（看吧，小技巧~）
>***2.看它的下载方法里面是怎么实现的***

```
//通过url创建一个Operation
- (id <SDWebImageOperation>)loadImageWithURL:(nullable NSURL *)url
                                     options:(SDWebImageOptions)options
                                    progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                   completed:(nullable SDInternalCompletionBlock)completedBlock {
    // Invoking this method without a completedBlock is pointless
    //断言...
    NSAssert(completedBlock != nil, @"If you mean to prefetch the image, use -[SDWebImagePrefetcher prefetchURLs] instead");

    // Very common mistake is to send the URL using NSString object instead of NSURL. For some strange reason, Xcode won't
    // throw any warning for this type mismatch. Here we failsafe this error by allowing URLs to be passed as NSString.
    //如果传入的url是字符串类型，则将url转换成NSURL类型(容错处理)
    if ([url isKindOfClass:NSString.class]) {
        url = [NSURL URLWithString:(NSString *)url];
    }

    // Prevents app crashing on argument type error like sending NSNull instead of NSURL
    //如果经过上述处理后的url还是不是NSURL类型，那么将url设置为nil.
    if (![url isKindOfClass:NSURL.class]) {
        url = nil;
    }

    __block SDWebImageCombinedOperation *operation = [SDWebImageCombinedOperation new];
    __weak SDWebImageCombinedOperation *weakOperation = operation;

    BOOL isFailedUrl = NO;
    if (url) {
        //创建一个互斥锁防止，保证线程安全
        @synchronized (self.failedURLs) {
            //判断url是否是下载失败过,
            isFailedUrl = [self.failedURLs containsObject:url];
        }
    }
    //如果url不存在那么直接返回一个block，如果url存在那么继续
    //如果options的值没有设置为失败后重试并且url下载失败过,执行完成block返回错误
    if (url.absoluteString.length == 0 || (!(options & SDWebImageRetryFailed) && isFailedUrl)) {
        [self callCompletionBlockForOperation:operation completion:completedBlock error:[NSError errorWithDomain:NSURLErrorDomain code:NSURLErrorFileDoesNotExist userInfo:nil] url:url];
        return operation;
    }
    //将operations加入到runningOperations数组中
    @synchronized (self.runningOperations) {
        [self.runningOperations addObject:operation];
    }
    //根据url获取到对应的key
    NSString *key = [self cacheKeyForURL:url];
    //根据key异步在缓存中查找是否有图片，且返回一个operation
    operation.cacheOperation = [self.imageCache queryCacheOperationForKey:key done:^(UIImage *cachedImage, NSData *cachedData, SDImageCacheType cacheType) {
        //如果operation取消了,将它从self.runningOperations中移除
        if (operation.isCancelled) {
            [self safelyRemoveOperationFromRunning:operation];
            return;
        }
        //条件1.在缓存中没有找到图片或者options设置为了SDWebImageRefreshCached
        //条件2.代理允许下载，或者代理不响应imageManager:shouldDownloadImageForURL:方法
        if ((!cachedImage || options & SDWebImageRefreshCached) && (![self.delegate respondsToSelector:@selector(imageManager:shouldDownloadImageForURL:)] || [self.delegate imageManager:self shouldDownloadImageForURL:url])) {
            //如果在缓存中找到了图片，但是需要更新缓存
            if (cachedImage && options & SDWebImageRefreshCached) {
                // If image was found in the cache but SDWebImageRefreshCached is provided, notify about the cached image
                // AND try to re-download it in order to let a chance to NSURLCache to refresh it from server.
                //传递缓存中的image，同时尝试重新下载它来让NSURLCache有机会接收服务器端的更新
                [self callCompletionBlockForOperation:weakOperation completion:completedBlock image:cachedImage data:cachedData error:nil cacheType:cacheType finished:YES url:url];
            }

            // download if no image or requested to refresh anyway, and download allowed by delegate
            //设置downloaderOptions
            SDWebImageDownloaderOptions downloaderOptions = 0;
            if (options & SDWebImageLowPriority) downloaderOptions |= SDWebImageDownloaderLowPriority;
            if (options & SDWebImageProgressiveDownload) downloaderOptions |= SDWebImageDownloaderProgressiveDownload;
            if (options & SDWebImageRefreshCached) downloaderOptions |= SDWebImageDownloaderUseNSURLCache;
            if (options & SDWebImageContinueInBackground) downloaderOptions |= SDWebImageDownloaderContinueInBackground;
            if (options & SDWebImageHandleCookies) downloaderOptions |= SDWebImageDownloaderHandleCookies;
            if (options & SDWebImageAllowInvalidSSLCertificates) downloaderOptions |= SDWebImageDownloaderAllowInvalidSSLCertificates;
            if (options & SDWebImageHighPriority) downloaderOptions |= SDWebImageDownloaderHighPriority;
            if (options & SDWebImageScaleDownLargeImages) downloaderOptions |= SDWebImageDownloaderScaleDownLargeImages;
            
            if (cachedImage && options & SDWebImageRefreshCached) {
                // force progressive off if image already cached but forced refreshing
                downloaderOptions &= ~SDWebImageDownloaderProgressiveDownload;
                // ignore image read from NSURLCache if image if cached but force refreshing
                downloaderOptions |= SDWebImageDownloaderIgnoreCachedResponse;
            }
            //到这里才真的去下载图片...
            SDWebImageDownloadToken *subOperationToken = [self.imageDownloader downloadImageWithURL:url options:downloaderOptions progress:progressBlock completed:^(UIImage *downloadedImage, NSData *downloadedData, NSError *error, BOOL finished) {
                __strong __typeof(weakOperation) strongOperation = weakOperation;
                //如果操作取消,do nothing
                if (!strongOperation || strongOperation.isCancelled) {
                    // Do nothing if the operation was cancelled
                    // See #699 for more details
                    // if we would call the completedBlock, there could be a race condition between this block and another completedBlock for the same object, so if this one is called second, we will overwrite the new data
                }
                //如果有错误，将error回调出去
                else if (error) {
                    
                    [self callCompletionBlockForOperation:strongOperation completion:completedBlock error:error url:url];

                    if (   error.code != NSURLErrorNotConnectedToInternet
                        && error.code != NSURLErrorCancelled
                        && error.code != NSURLErrorTimedOut
                        && error.code != NSURLErrorInternationalRoamingOff
                        && error.code != NSURLErrorDataNotAllowed
                        && error.code != NSURLErrorCannotFindHost
                        && error.code != NSURLErrorCannotConnectToHost
                        && error.code != NSURLErrorNetworkConnectionLost) {
                        @synchronized (self.failedURLs) {
                            //将url添加到失败列表中、
                            [self.failedURLs addObject:url];
                        }
                    }
                }
                else {
                    //如果设置了下载失败重试，则将url从列表中移除、
                    if ((options & SDWebImageRetryFailed)) {
                        @synchronized (self.failedURLs) {
                            [self.failedURLs removeObject:url];
                        }
                    }
                    
                    BOOL cacheOnDisk = !(options & SDWebImageCacheMemoryOnly);
                    //设置了刷新缓存、有缓存、下载图片失败
                    if (options & SDWebImageRefreshCached && cachedImage && !downloadedImage) {
                        // Image refresh hit the NSURLCache cache, do not call the completion block
                    }
                    //图片下载成功，且设置了图片变形，代理也能响应变形方法
                    else if (downloadedImage && (!downloadedImage.images || (options & SDWebImageTransformAnimatedImage)) && [self.delegate respondsToSelector:@selector(imageManager:transformDownloadedImage:withURL:)]) {
                       //全局队列异步执行。
                        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
                            //调用代理方法完成图片的transform
                            UIImage *transformedImage = [self.delegate imageManager:self transformDownloadedImage:downloadedImage withURL:url];
                            //对已经transform的图片进行缓存
                            if (transformedImage && finished) {
                                BOOL imageWasTransformed = ![transformedImage isEqual:downloadedImage];
                                // pass nil if the image was transformed, so we can recalculate the data from the image
                                [self.imageCache storeImage:transformedImage imageData:(imageWasTransformed ? nil : downloadedData) forKey:key toDisk:cacheOnDisk completion:nil];
                            }
                            //执行回调
                            [self callCompletionBlockForOperation:strongOperation completion:completedBlock image:transformedImage data:downloadedData error:nil cacheType:SDImageCacheTypeNone finished:finished url:url];
                        });
                    } else {
                        //如果不需要做图片的变形，而且图片下载完成，那么就直接缓存
                        if (downloadedImage && finished) {
                            [self.imageCache storeImage:downloadedImage imageData:downloadedData forKey:key toDisk:cacheOnDisk completion:nil];
                        }
                        //执行完成回调
                        [self callCompletionBlockForOperation:strongOperation completion:completedBlock image:downloadedImage data:downloadedData error:nil cacheType:SDImageCacheTypeNone finished:finished url:url];
                    }
                }
                //如果已经完成下载了，将operation移除
                if (finished) {
                    [self safelyRemoveOperationFromRunning:strongOperation];
                }
            }];
            
            @synchronized(operation) {
                // Need same lock to ensure cancelBlock called because cancel method can be called in different queue
                operation.cancelBlock = ^{
                    [self.imageDownloader cancel:subOperationToken];
                    __strong __typeof(weakOperation) strongOperation = weakOperation;
                    [self safelyRemoveOperationFromRunning:strongOperation];
                };
            }
            
        }
        //其他情况：代理不允许下载，或者是没有设置更新缓存
        else if (cachedImage) {
            __strong __typeof(weakOperation) strongOperation = weakOperation;
            //执行完成回调
            [self callCompletionBlockForOperation:strongOperation completion:completedBlock image:cachedImage data:cachedData error:nil cacheType:cacheType finished:YES url:url];
            //移除operation
            [self safelyRemoveOperationFromRunning:operation];
        }
        //图片既没有在缓存中找到，代理也不允许下载
        else {
            // Image not in cache and download disallowed by delegate
            __strong __typeof(weakOperation) strongOperation = weakOperation;
            //执行完成回调
            [self callCompletionBlockForOperation:strongOperation completion:completedBlock image:nil data:nil error:nil cacheType:SDImageCacheTypeNone finished:YES url:url];
            //移除operation
            [self safelyRemoveOperationFromRunning:operation];
        }
    }];

    return operation;
}
```
看上去代码量有点多，逻辑很复杂，其实确实也是这样的~（哈哈，我刚看的时候，脑壳都要绕晕了~）。不过整理下发现，其实差不多也就是做了两件事：
>1. 通过url对应的key去缓存里找有没有对应的图片；
>2. `self.imageDownloadem`调用下载方法去下载图片。

然后要补充的几点是：
* ***SDWebImageCombinedOperation*** 

```
@interface SDWebImageCombinedOperation : NSObject <SDWebImageOperation>

@property (assign, nonatomic, getter = isCancelled) BOOL cancelled;
@property (copy, nonatomic, nullable) SDWebImageNoParamsBlock cancelBlock;
@property (strong, nonatomic, nullable) NSOperation *cacheOperation;

@end

@implementation SDWebImageCombinedOperation

- (void)setCancelBlock:(nullable SDWebImageNoParamsBlock)cancelBlock {
    // check if the operation is already cancelled, then we just call the cancelBlock
    if (self.isCancelled) {
        if (cancelBlock) {
            cancelBlock();
        }
        _cancelBlock = nil; // don't forget to nil the cancelBlock, otherwise we will get crashes
    } else {
        _cancelBlock = [cancelBlock copy];
    }
}

- (void)cancel {
    @synchronized(self) {
        self.cancelled = YES;
        if (self.cacheOperation) {
            [self.cacheOperation cancel];
            self.cacheOperation = nil;
        }
        if (self.cancelBlock) {
            self.cancelBlock();
            self.cancelBlock = nil;
        }
    }
}

@end
```
它有三个属性，一个是否取消的`Bool`属值，一个取消``Bool`和一个缓存`operation`，很明显，我们可以通过它来取消下载`operation`。

在上面的下载方法中，有一个`__block`修饰的`operation`和一个`__weak`修饰的w`eakOperation`，

```
__block SDWebImageCombinedOperation *operation = [SDWebImageCombinedOperation new];
__weak SDWebImageCombinedOperation *weakOperation = operation;
```
我们说下***__block*** 和 ***__weak***的区别:
`__block`用于指明当前声明的变量被`block`捕获后，可以在`block`中改变变量的值。因为在`block`声明的同时会捕获该`block`所使用的全部自动变量的值，这些值在`block`中只有“使用权”而没有“修改权”。而用`__block`修饰的变量，则可以在`block`改变变量的值了。同时，`__block`并不是弱引用，如果有循环引用的话，我们需要在`block`内部手动释放，将其设置为`nil`。
`__weak`是所有权修饰符，`__weak`本身是可以避免循环引用的，我们在`block`内部使用的时候，需要用`__strong`来强引用，防止变量提前释放掉了。

##### 4.在缓存中查找图片:queryCacheOperationForKey
这个方法是`SDImageCache`里面定义的一个方法，我们开方法的实现:

```
- (nullable NSOperation *)queryCacheOperationForKey:(nullable NSString *)key done:(nullable SDCacheQueryCompletedBlock)doneBlock {
    //如果key为空，回调完成block,return
    if (!key) {
        if (doneBlock) {
            doneBlock(nil, nil, SDImageCacheTypeNone);
        }
        return nil;
    }

    // First check the in-memory cache...
    //首先通过key在内存的缓存中查找有没有图片...
    UIImage *image = [self imageFromMemoryCacheForKey:key];
    //如果有内存缓存中有图片
    if (image) {
        NSData *diskData = nil;
        //如果gif图片(非动图)
        if (image.images) {
            //通过key找到磁盘中图片的data
            diskData = [self diskImageDataBySearchingAllPathsForKey:key];
        }
        //执行完成block
        if (doneBlock) {
            doneBlock(image, diskData, SDImageCacheTypeMemory);
        }
        return nil;
    }

    //如果内存中没有，则在磁盘中查找，如果找到，则将其放在内存缓存中，并调用doneBlock
    NSOperation *operation = [NSOperation new];
    //在ioQueue中串行处理所有磁盘缓存
    dispatch_async(self.ioQueue, ^{
        if (operation.isCancelled) {
            // do not call the completion if cancelled
            return;
        }
        //创建自动释放池，内存及时释放
        @autoreleasepool {
            //根据key去磁盘中找到图片的data数据
            NSData *diskData = [self diskImageDataBySearchingAllPathsForKey:key];
            //根据图片的url的对应的key去磁盘缓存中查找图片
            UIImage *diskImage = [self diskImageForKey:key];
            //如果磁盘中有图片，并且设置了内存缓存机制，再将图片缓存在内存中
            if (diskImage && self.config.shouldCacheImagesInMemory) {
                NSUInteger cost = SDCacheCostForImage(diskImage);
                //用NSCache将图片存储到内存中
                [self.memCache setObject:diskImage forKey:key cost:cost];
            }

            if (doneBlock) {
                //主线程执行完成block
                dispatch_async(dispatch_get_main_queue(), ^{
                    doneBlock(diskImage, diskData, SDImageCacheTypeDisk);
                });
            }
        }
    });

    return operation;
}
```
方法中用到了一个串行队列`self.ioQueue`，它的声明和初始化是这样的：

```
@property (SDDispatchQueueSetterSementics, nonatomic, nullable) dispatch_queue_t ioQueue;
// Create IO serial queue
ioQueue = dispatch_queue_create("com.hackemist.SDWebImageCache", DISPATCH_QUEUE_SERIAL);
```

### 总结

到目前为止，只进行到了通过key去缓存中查找有没有图片，在完成回调中，我们经过一些逻辑处理，到最后面才去用`self.imageDownloader`去调用`download`方法。下篇我们可能就会去了解下`download`方法是如何下载的，网络是怎么处理的。

（PS,有很多理解不到位的地方，还希望和大家一起交流学习~）


