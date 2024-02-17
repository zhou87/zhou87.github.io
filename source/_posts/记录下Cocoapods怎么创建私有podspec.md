---
title: 记录下Cocoapods怎么创建私有podspec
comments: true
categories: iOS
date: 2019-03-13 18:45:08
tags: 
    - Cocoapods
    - 记录
---

### 前言
`Cocoapods`相信大家都不陌生，应该是目前非常主流的iOS的包管理工具了，这样也省去了我们手动导入第三方库的麻烦。当然，这个库使用起来也很简单，`pod init`,`pod install`,`pod update`,基本上三板斧完全可以应付日常开发，如此简洁，让`iOS`开发者们青睐也是情有可原的。但是，如果你是个有点追求的开发，平时肯定会去写点自己的小东西，发布到各大平台为广大开发者们使用。这是就需要用到`cocoapods`来创建自己的库了，其实也没有什么高大上的地方，只要多用两次基本上就会了。

下面就简单记录下怎么建自己的私有仓库。

<!--more-->
### 创建私有组件库

`pod lib create HPTestPod`

打开终端，在想要建项目的目录下面执行下面这句代码，终端会询问你五个问题：

`1.What platform do you want to use?? [ iOS / macOS ]`

你想在哪个平台上使用？当然各取所需，一般选iOS，那么在终端输入`iOS`即可。

`2.What language do you want to use?? [ Swift / ObjC ]`

你用的是哪门子的语言？同上

`3.Would you like to include a demo application with your library? [ Yes / No ]`

你希望在项目里面建一个demo吗？ 有个demo可能更好的帮组我们测试代码，个人觉得很有用，一般都会选`Yes`。

`4.Which testing frameworks will you use? [ Quick / None ]`

你想用哪个测试框架？距离的测试框架我也没怎么用过，有兴趣的大佬们可以研究下，可选可不选。

`5.Would you like to do view based testing? [ Yes / No ]`

是否基于View测试？ 如果对测试要求不是很高的话可以选择`No`。

做完这五道选择题之后，`cocoapods`会自动创建好项目，并执行`pod install`安装好依赖包。就是这么简单，一行命令就可以创建好项目。项目创建好之后会自动打开项目，有时候我们也可以`cd`到刚刚创建好的项目的`Example`目录下，执行`open HPTestPod.xcworkspace`打开项目。我们要做的是修改`podSpec`文件，自动生成的`podSpec`文件有很多注释，还有一些描述性的东西需要我们改一下，要不然在`push`到`repo`的时候回有警告，导致`push`不上去。我们先看看这个文件出生时的样子吧：

```
#
# Be sure to run `pod lib lint TestPod.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see https://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = 'TestPod'
  s.version          = '0.1.0'
  s.summary          = 'A short description of TestPod.'

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!

  s.description      = <<-DESC
TODO: Add long description of the pod here.
                       DESC

  s.homepage         = 'https://github.com/LYMelody/TestPod'
  # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'LYMelody' => 'zhouhuiping@souche.com' }
  s.source           = { :git => 'https://github.com/LYMelody/TestPod.git', :tag => s.version.to_s }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.ios.deployment_target = '8.0'

  s.source_files = 'TestPod/Classes/**/*'
  
  # s.resource_bundles = {
  #   'TestPod' => ['TestPod/Assets/*.png']
  # }

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3'
end
```

这是一个用`ruby`写的脚本文件，当然我们大可不必去大刀阔斧地去啃`ruby`,知道一些平时要用到的地方基本就好了。
`s.name`: 当然就是这个`pod`的名字，
`s.version`: 是这个库的版本，
`s.summary`: 这个`pod`的简要概括。
`s.description`: 对这个库的一个稍微详细的描述，也可以很短，**但是不能比summary短，不然后面就会有警告**，
`s.homepage`: 项目主页，如果是`github`账号创建的话基本上也没什么变动，
`s.source`: 项目仓库地址，待会我们需要创建一个远程仓库，
`s.ios.deployment_target`: 最低`iOS`版本，
`s.source_files`: `pod`的源文件，项目里面的文件基本上都放`TestPod/Classes/**/*`下面。
这里我们需要加一个: `s.swift_version = '4.0'`，不然的话在`lint`的时候也会报警告,因为`cocoapods`默认的`swift`版本是`3.2`，需要我们指定下`swift`版本。大概这样就差不多了，然后我们可以执行下`pod lib lint`,
检查下`spec`文件是否符合规则,经过上面的一些改动基本上没啥问题，一般输出如下：

```
-> flowTestPod (0.1.0)

flowTestPod passed validation.
```

接下来我们需要去`GitHub`上建一个仓库放我们的私有库。有了仓库地址之后就可以把我们刚才的代码直接推到远程了，在当前目录下执行：

```
git add .
git commit -m "Init pod"
git remote add origin git@github.com:LYMelody/TestPod.git
git push origin master
```

因为`podspec`文件中获取`Git`版本控制的项目还需要`tag`号，所以我们要打上一个`tag`：

```
git tag -m "first release" 0.1.0
git push origin master --tags
```

建立了远程仓库还不够，我们还需要建一个存放`podspec`文件的库。同样在`github`上新建一个仓库，比如`HPSpecs`，刚好最近微软开放了`github`私有库的权限，我们可以把这个建立为自己的私有仓库。建好之后，拿到仓库地址我们可以直接将`podspec`文件提交到`spec repo`。也很简单：

```
pod repo push HPSpecs TestPod.podspec
```

这样，这个组件库就添加到了我们私有库`HPSpecs`中了，可以在~`/.cocoapods/repos/HPSpecs`目录下看到我们的库和`spec`文件。远程仓库也能看到这次提交。还不确定是否只做成功的话，可以`pod search TestPod`查看刚提交的pod信息。我们也可以在其他项目中导入`source`使用这个`pod`了。

这里需要提下的是，当你发版的时候需要将`spec`的版本改成你要发的对应的版本，记得打上`tag`，并确保与`spec`上面的`tag`一直。然后可以直接执行：

```
pod repo push HPSpecs TestPod.podspec
```

更新pod。

####参看文章
[使用Cocoapods创建私有podspec](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)



