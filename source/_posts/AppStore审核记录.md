---
title: AppStore审核记录
comments: true
categories: iOS
date: 2017-11-08 09:04:21
tags:
	- iOS App Store审核
keywords:
---

### 记录一下上传AppStore出现的一些问题
>1.公司名混淆

```java
Guideline 5.2.1 - Legal - Intellectual Property


The seller and company names associated with your app do not reflect financial institution, as required by Guideline 5.2.1 of the App Store Review Guidelines.

Next Steps

Your app must be published under a seller name and company name that reflects financial institution. If you have developed this app on behalf of a client, please advise your client to add you to the development team of their Apple Developer account.

Once created, you cannot change your seller name or company name in iTunes Connect. For assistance with changing your company name or seller name, you will need to contact iTunes Connect through the Contact Us page. Select Getting Started from the first dropdown menu, then select General iTunes Connect Inquiry to contact the appropriate iTunes Connect team.
```
<!-- more -->
附带几张截图：
![](/图片测试/attachment-8757227818614007228Screenshot-1107-135913.png)
![](/图片测试/attachment-1902747273820313655Screenshot-1107-135537.png)
![](/图片测试/attachment-3918554658723829359Screenshot-1107-135546.png)
![](/图片测试/attachment-5135181536681855368Screenshot-1107-135917.png)


***解决办法:*** 出现这种类类型的错误，一般都是APP中使用了第三方产品的logo或者是图片之类的并未获取许可，特别是金融类的App，非本公司产品相关的敏感字眼，logo和图片。当然，也可以通过后台来控制，审核的时候我们部署简单点的内容，上线之后再把一些可能过不了的内容上上去。我们现在就是这样实现的。


