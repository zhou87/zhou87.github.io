---
title: SCCSwiftNetWork是怎么封装的
comments: true
date: 2019-01-22 09:00:32
categories: iOS
tags:
    - 归纳总结
    - 源码阅读
---
### 前言
上次罗列了一下在`App`里面用到的一些网络请求方式，概括的说应该是两总类型：一类是直接`url`各种语法糖链接起来拿到请求数据，另一类就是针对`RX`做了一层封装，用`RX`的方式调用。那么这里我们就来总结一下那些类似`"post"`、`"vaildData"`之类的关键字里面做了什么，其他的方法又是怎么封装的。

### 从一个网络请求说起

先来看一个简单的网络请求吧：

```
let url = "https://www.baidu.com/" (这里是需要发送请求的url)
        
let params = [
    "name":"tangeche",
    "phone":"12312312",
    "token":"YEHodfIOU",
    "code": "123456"]
    
url
    .headers(["Content-Type" : "application/json"])
    .timeout(5)
    .parameters(params)
    .post(encoding: SCCParamterEncoding.jsonRaw)
    .validData
    .subscribe(onNext: { (json) in
        print(json)
    }, onError: { (error) in
        print(error)
    })
    .disposed(by: bag)
```

<!-- more -->

这里的`url`根据类型推断应该是一个`String`类型，后面直接`.headers`,很明显是对`String`做了扩展。点进去看就知道，在`Alamofire`里面有对`String`做了一个扩展：

```
extension String: URLConvertible {
    public func asURL() throws -> URL {
        guard let url = URL(string: self) else { throw AFError.invalidURL(url: self) }
        return url
    }
}
```
然后，`SCCSwiftNetWork`里面又对`URLConvertible`做了一个扩展:

```
extension URLConvertible {

    public func timeout(_ timeout: TimeInterval) -> SCCSwiftyNetwork.SCCURL

    public func validatableParamters(_ params: SCCModelKeeper.SCCValidatableParameters) -> SCCSwiftyNetwork.SCCURL

    public func parameters(_ params: Parameters) -> SCCSwiftyNetwork.SCCURL

    public func headers(_ headers: HTTPHeaders) -> SCCSwiftyNetwork.SCCURL

    public func download(to path: String) -> RxSwift.Observable<String>
}
```
这样我们就能直接拿一个`String`做上面那些操作了，比如`timeout`是设置网络超时时间，`validatableParamters`是对传入的参数做一些数据结构校验，也是大佬们封装好的，`parameters`显而易见是上传参数，`headers`请求头，`download`是一个将数据下载到本地的方法。但是，有没有注意到这些扩展方法里面都返回的是一个`SCCURL`，那这个`SCCURL`又是什么呢？

点开`SCCURL`类你会发现，原来他也是扩展自`URLConvertible`，那么他的实例照样可以`.headers`、`.parameters`了。正如你看到的，前面这些都是在配置请求前的参数之类的，那么这些准备做好了是不是要开始发送网络请求了。没错，`SCCURL`里面就扩展了发送网络请求的方法：

```
public extension SCCURL {
    
    func get(encoding: ParameterEncoding = URLEncoding.default) -> Observable<(Any, String)> {
        return request(method: .get, encoding: encoding)
    }
    
    func post(encoding: ParameterEncoding = URLEncoding.default) -> Observable<(Any, String)> {
        return request(method: .post, encoding: encoding)
    }
    
    ...
        
    func request(method: HTTPMethod, encoding: ParameterEncoding) -> Observable<(Any, String)> {
        return request(method: method, parameters: parameters, headers: headers, timeout: timeout, encoding: encoding)
    }
    ...
}
```
这里的扩展应该涵盖了大部分的请求方式，主流的请求方式应该还是
`get`,`post`，正如例子中用到的`post`。最后，所有的请求都会归纳到方法，不妨把这个方法直接贴出来：

```
// Instance member cannot be uesed as default parameter
    func request(method: HTTPMethod,
             parameters: Parameters?,
                headers: HTTPHeaders?,
                timeout: TimeInterval? = nil,
               encoding: ParameterEncoding = URLEncoding.default,
               useCache: Bool = false) -> Observable<(Any, String)> {
        
        var headers = headers
        let url = (try? self.asURL())?.absoluteString ?? ""
        return Observable<(Any, String)>.create { observer in
            /// 一个判断是否有缓存的Bool值
            var hasCache = false
            /// 判断是否有缓存，如果有直接取缓存的数据返回
            if useCache, let value = NSKeyedUnarchiver.unarchiveObject(withFile: self.cacheKey) {
                observer.onNext((value, url))
                hasCache = true
            }

            if var commonHeader = RequestCommonConfiguration.shared.headers {
                /// 更新请求headers
                headers?.forEach({ commonHeader[$0] = $1 })
                headers = commonHeader
            }

            // Configure timeout for EscapedDog Dalao
            /// 如果有设置超时时间，根据超时时间得到一个sessionManager
            let manager: SessionManager
            if let timeout = timeout, timeout > 0 {
                if let cachedManager = customManagers[timeout] {
                    manager = cachedManager
                } else {
                    manager = configureManager(with: timeout)
                    // Retain manager
                    customManagers[timeout] = manager
                }
            } else {
                manager = defaultManager
            }

            let task = manager.request(self, method: method, parameters: parameters, encoding:encoding, headers: headers)
                .responseJSON { response in
                    /// 如果抛出错误，上报错误
                    if let error = response.result.error {
                        SCCValidateResultReporter.request(url: url, errors: [], error: "\(error.code()) \(error.formatterDescription())")
                        observer.onError(error)
                        return
                    }

                    if let value = response.result.value {
                        /// 如果使用缓存的话，在之前没有缓存的情况下将数据写入缓存,有则不写
                        if useCache {
                            _ = NSKeyedArchiver.archiveRootObject(value, toFile: self.cacheKey)
                            if hasCache {
                                observer.onCompleted()
                                return
                            }
                        }
                        /// 如果不使用缓存，将返回的数据和url构成一个元组返回
                        observer.onNext((value, url))
                    }
                    observer.onCompleted()
            }
            return Disposables.create(with: task.cancel)
        }
    }
```
可以看到，这个方法发送了一个请求，返回了一个`(Any, String)`类型的的`observable`。说下几个值得注意的点：

>* **缓存处理**

在传入参数中有个`useCache`(不过外部好像并没有用到过),如果使用缓存的话，那么会先根据`url`创建一个`cacheKey`尝试获取本地有没有缓存上次请求的数据，如果有的话，`observer`直接将得到的缓存发送出去，标记`hasCache=true`。在请求成功后，同样是使用缓存的情况下根据`url`创建的`cacheKey`将请求回来的数据写入缓存，已有缓存的话，`observer`发送一个`completed`事件。

```
private extension URLConvertible {
    var cacheKey: String {
        let url = try? asURL()
        assert(url != nil, "Invalid URL")
        let cachePath = NSSearchPathForDirectoriesInDomains(.cachesDirectory, .userDomainMask, true)[0]
        let fileName = url!.absoluteString.replacingOccurrences(of: "/", with: "-")
        return "\(cachePath)/\(fileName)"
    }
}
```

>* **默认headers**

为了方便服务端获取更多的有关`App`的信息，单独写了一个类
`RequestCommonConfiguration`。这是一个单例，里面为
`headers`写死了一些默认的App的基本数据，类似`Appversion`、
`AppName`等等。发送请求前，再对外面传进来的`headers`做了一次拼接。

>* **根据timeout创建SessionManager**


`customManager`是一个`[TimeInterval:SessionManager]`键值对，根据不同的
`timeout`时间维护不同的`SessionManager`。这里为什么需要根据`timeout`来维护不同的`session`而不用一个全局的
`session`，大概可以理解为如果公用一个`session`的话那当前一个请求还没有真正发出去的时候，下一个请求来了我们改变了`session`的`timeout`那么会影响到前一个请求。

>* **错误上报**

```
SCCValidateResultReporter.request(url: url, errors: [], error: "\(error.code()) \(error.formatterDescription())")
```
是一个封装好的将错误上报到一个错误统计平台，便于App的错误分析。


### 解析数据

请求发出去了，数据也拿到了，那么接下来就是解析数据了。前面我们已经知道网络请求回来返回的是一个`(Any, String)`类型的`Observable`,在数据解析文件里面对这个类型的`Observable`做这么几个方法的扩展：

```
public extension Observable where Element == (Any, String) {

    var validDataUrl: Observable<(JSON, String)> {
        return flatMap(parse)
    }
       
    var validData: Observable<JSON> {
        return map{ $0.0 }
            .flatMap(parseOnlyObject)
    }
}
```
>* **validDataUrl**

可以看到这里的`validDataUrl`属性是将`Observable<(Any,String)>`转化为了`Observable<(JSON,String)>`,通过一个`parse`方法。

```
private func parse(object: (Any, String)) -> Observable<(JSON, String)> {
    let json = JSON(object.0)
    guard let _ = json.dictionary else {
        let error = SCCError.invalidFormat(message: Message.invalidFormat, object: object.0)
        SCCValidateResultReporter.request(url: object.1, errors: [], error: "\(error.code()) \(error.formatterDescription())")
        return .error(error)
    }
    return handleJSON(json: json, url: object.1)
}
```
这里还看不出来对数据做了怎么的操作，只是将`Any`类型的数据转化为了`JSON`，如果有`dictionary`值则继续处理，否则发送一个错误事件并将错误上报。那我们再看看`handleJSON`这个方法是怎么实现的：

```
private func handleJSON(returnKey: String = Key.data, json: JSON, url: String) -> Observable<(JSON, String)> {
    /// 如果返回'success'字段为true
    if check(json: json[Key.success]) {
        if returnKey.isEmpty {
            return .just((json, url))
        }
        return .just((json[returnKey], url))
    } else {
        
        let error = handleError(json: json)
        SCCValidateResultReporter.request(url: url, errors: [], error: "\(error.code()) \(error.formatterDescription())")
        return .error(error)
    }
}
```
这个方法先对传进来的`json`的`success`字段做一次校验，如果为`false`的话直接发送错误事件并上报。如果为`ture`的话，这里的`returnKey`这个参数应该是一个取服务端对应`key`里面的数据，目前默认约定好的应该是`data`(`Key`是一个结构体，里面有很多静态属性，data = "data")里面的数据。这个最好应该是跟服务端约定好的事，可以不一定是`data`。目前看项目里面别的地方好像都没有传入这个参数，不过加一个这个参数可以保证灵活性吧，万一哪天一个新来的服务端大佬不熟悉这个规则，将`data`变为`Data`那还是有一定的补救的余地，不过这中情况也是服务端可以直接修复的，也不需要客户端来补救。

>* **validData**

对应的，`validData`是将`Observable<(Any,String)>`转化为`Observable<JSON>`，这里先用`map`将`(Any,String)`转化为`Any`,再`flatMap`调用`parseOnlyObject`：

```
private func parseOnlyObject(object: Any) -> Observable<JSON> {
    let json = JSON(object)
    guard let _ = json.dictionary else {
        return .error(SCCError.invalidFormat(message: Message.invalidFormat, object: object))
    }
    return handleJSON(json: json)
}
```
这里和上面和类似，就不用过多笔墨了。可以看一下`handleError`这个方法怎么实现的：

```
func handleError(json: JSON) -> SCCError {
    
    let code = json[Key.code].intValue
    if ErrorCode.notLogin.rawValue == code || ErrorCode.tokenLost.rawValue == code {
        NotificationCenter.default.post(name: .SCCNotLogin, object: nil)
    }
    let traceId = json[Key.traceId].stringValue
    var message = "\(json[Key.msg].stringValue)\n\(traceId)"
    if !traceId.isEmpty, let topVC = UIViewController.applicationTopVC() {
        SCCModulor.sharedInstance().moduleName("wirelessToast", openWithParams: ["icon": "qrcode", "text": message, "qrcodeText": traceId, "vc": topVC, "duration": "2000"], callback:nil)
        //已经弹过toast，防止上层业务二次弹出，将message置空，ui toast层做判断拦截
        message = ""
    }
    let error = SCCError.business(errorCode: code,
                                  message: message,
                                  object: json[Key.data])
    return error
}
```
`SCCError`是一个对`App`网络错误类别统一归类的的类，这里的业务逻辑是如果是未登录或者`token`丢失的错误的话会发送一个未登录通知。另一个逻辑是在有`traceId`的情况会找到当前`App`的`topVC`弹一个带有二维码的`toast`。这个找`topVc`的方法值得学习下：

```
extension UIViewController {
    
    static func applicationTopVC() -> UIViewController? {
        var window: UIWindow? = UIApplication.shared.keyWindow
        if window?.windowLevel != UIWindowLevelNormal {
            let windows = UIApplication.shared.windows
            for tmpWin: UIWindow in windows {
                if tmpWin.windowLevel == UIWindowLevelNormal {
                    window = tmpWin
                    break
                }
            }
        }
        for frontView: UIView? in window?.subviews ?? [UIView?]() {
            var nextResponder = frontView?.next
            if let lenderClass = objc_getClass("UILayoutContainerView") as? UIView {
                
                if let isMember = frontView?.isMember(of: lenderClass.classForCoder), !isMember {
                    let arr = frontView?.value(forKey: "subviewCache") as? [Any]
                    if (arr?.count ?? 0) > 0 {
                        let v = arr?[0] as? UIView
                        nextResponder = v?.next
                    } else {
                        nextResponder = frontView?.subviews[0].next
                    }
                }
            }
       
            if (nextResponder is UITabBarController) {
                let tabbar = nextResponder as? UITabBarController
                if let selectedIndex = tabbar?.selectedIndex {
                    let vc = tabbar?.viewControllers?[selectedIndex]
                    guard let nav = vc as? UINavigationController else {
                        return vc
                    }
                    return nav.childViewControllers.last
                }
            } else if (nextResponder is UINavigationController) {
                let nav = nextResponder as? UINavigationController
                return nav?.viewControllers.last
            } else if (nextResponder is UIViewController) {
                let vc = nextResponder as? UIViewController
                return vc
            }
        }
        return nil
    }
}
```
到此，第一类请求方式差不多就过完了，接下来我们主要看看怎么对这些方法进行`RX`方法的封装。

### API+RX

跟第一类方式一样，先看一个简单的网络请求：

```
BankResource()
    .rx.jsonWithParams()
    .subscribe({ (event) in
        switch event {
        case .next(_):
            break
        case .error(let error):
            self.scc.toast(content: error.formatterDescription())
            break
        default: break
        }
    })
    .disposed(by: bag)
```
分析这个方法之前我们先看一个类`SCCApi`：

```
open class SCCApi: NSObject, APIType {

    public lazy var url: SCCURL = {
        return SCCURL(url: self.apiURL())
    }()

    public lazy var method: HTTPMethod = {
        return self.apiType()
    }()

    open func apiURL() -> String! {
        assertionFailure("Should be overrided by subclass.")
        return ""
    }

    open func apiType() -> HTTPMethod {
        return .get
    }

    open func composeJSONWithPropertyParams() -> Observable<JSON> {
        return observableData.json
    }

    open func composeJSONWithParams(_ params: Parameters = [:]) -> Observable<JSON> {
        return url.request(method: method, parameters: params, headers: headers).json
    }
}

postfix operator =>

public postfix func => <T: Mappable>(object: Any?) -> T? {
 
    return Mapper().map(JSONObject: object)
}

public postfix func => <T: Mappable>(object: Any?) -> [T]? {

    return Mapper().mapArray(JSONObject: object)
}
```
可以看到，这个类有两个属性：`url`,`method`;四个方法：`apiURL()`，`apiType()`，`composeJSONWithPropertyParams()`，`composeJSONWithParams(params:)`,刚看的时候可能会想`HTTPMethod`，`observableData`这些都是哪里来的，从没见过。嗯，别忘了它还遵守一个协议：`APIType`，我们看看里面都有些什么：

```
public protocol APIType {
    var url: SCCURL { get }
    var method: HTTPMethod { get }
    var headers: HTTPHeaders { get }
    var parameters: Parameters { get }
    var observableData: Observable<(Any, String)> { get }
}

public extension APIType {

    var method: HTTPMethod { return .get }
    var headers: HTTPHeaders {
        return RequestCommonConfiguration.shared.headers ?? [:]
    }
    
    var parameters: Parameters {
        var parameters: Parameters = [:]
        let mirror = Mirror(reflecting: self)
        mirror.children.forEach { parameters[$0.label!] = $0.value }
        return parameters
    }

    var observableData: Observable<(Any, String)> {
        return url.request(method: method, parameters: parameters, headers: headers)
    }
}
```
我们刚才疑惑的`observableData`在这里好像找到了，他是一个`Observable<(Any,String)>`,在扩展里面默认是根据默认参数调用了之前提到的网络请求方法。而`HTTPMethod`是一个`Alamofire`请求方式的类型别名:

```
public typealias Parameters = Alamofire.Parameters
public typealias HTTPHeaders = Alamofire.HTTPHeaders
public typealias HTTPMethod = Alamofire.HTTPMethod
```
还有`url`，`headers`，`parameters`这几个属性，可以留意一下`parameters`这个属性，它里面有用到`Mirror`(反射),这个很有意思。`Mirror(reflecting: self)`能够反射出这个实例的类型(`subjectType`)、属性集合(`children`)、对象展示类型(`displayStyle`)。这和`OC`里面`runtime`很类似，我们可以动态获取一个实例的属性了，正如上面代码里的一样。再倒回去看`SCCApi`，后面两个比较长的方法也就是将请求回来的数据做一次`JSON`(这里的json方法是老方法，应该用`vaildData`)解析，然后返回一个`Observable<JSON>`.这里有一个新版本的`SCCApi`:`SCCStandarApi`:

```
/// For compatibility with older version
open class SCCStandardApi: SCCApi {

    open override func composeJSONWithPropertyParams() -> Observable<JSON> {
        return observableData.validData
    }

    open override func composeJSONWithParams(_ params: Parameters) -> Observable<JSON> {
        return url.request(method: method, parameters: params, headers: headers).validData
    }
}
```
最后终于到了`RX`封装的方法了：

```
extension Reactive where Base: SCCApi {

    public func jsonWithPropertyParams() -> Observable<JSON> {
        return self.base.composeJSONWithPropertyParams()
    }

    public func jsonWithParams(_ params: Parameters = [:]) -> Observable<JSON> {
        return self.base.composeJSONWithParams(params)
    }
}

extension Reactive where Base: SCCApi {

    public func modelWithPropertyParams<T: Mappable>(_ modelType: T.Type, key: String? = nil) -> Observable<T?> {
        return base
            .rx
            .jsonWithPropertyParams()
            .map(valueForKey(key))
            .map(=>)
    }

    public func modelWithParams<T: Mappable>(_ params: Parameters = [:], modelType: T.Type, key: String? = nil) -> Observable<T?> {
        return base
            .rx
            .jsonWithParams(params)
            .map(valueForKey(key))
            .map(=>)
    }

    public func modelsWithPropertyParams<T: Mappable>(_ modelType: T.Type, key: String? = nil) -> Observable<[T]?> {
        return base
            .rx
            .jsonWithPropertyParams()
            .map(valueForKey(key))
            .map(=>)
    }

    public func modelsWithParams<T: Mappable>(_ params: Parameters = [:], modelType: T.Type, key: String? = nil) -> Observable<[T]?> {
        return base
            .rx
            .jsonWithParams(params)
            .map(valueForKey(key))
            .map(=>)
    }
}
```
这些方法分为两组，一组是返回`Observable<JSON>`，一类是返回一个泛型`model`。第一个方法不带参数，获取`api`的属性作为参数进行网络请求。第二个方法需要传入参数。
下面一组方法也是在将数据转为`JSON`之后再通过一层`Mapper`的转化，转化为`model`。`Mapper`方法是接入的第三方库`ObjectMapper`,这里就不展开讲了，下次有机会也对这个库做个总结。

### 结语

总体下来，这里对网络库的封装主要还是体现在对`Rx`的支持，这也更契合现在`swift`和`RX`结合使用的理念。还有就是对数据的校验和错误处理都比较严谨。这都是值得学习的地方。

这次`SCCSwiftNetWork`的总结就先到这里吧，有很多不到位的地方，也可能有很多理解或者是笔误的地方，请不吝赐教。谢谢！


