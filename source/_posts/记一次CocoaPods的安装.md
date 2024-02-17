---
title: 记一次CocoaPods的安装
comments: true
categories: iOS
date: 2017-11-02 17:36:35
tags: 
- CocoaPods
keywords:
---

### 前言
***
按照之前的一贯安装方法：

指定源：

```
$ gem sources --remove https://rubygems.org/
$ gem sources -a https://ruby.taobao.org/
$ sudo gem install cocoapods

```
结果报错如下：

```
ERROR:  Could not find a valid gem 'cocoapods' (>= 0), here is why:
          Unable to download data from https://ruby.taobao.org/ - server did not return a valid file (https://ruby.taobao.org/latest_specs.4.8.gz)
ERROR:  While executing gem ... (OpenSSL::SSL::SSLError)
    hostname "gems.ruby-china.org" does not match the server certificate
```

<!-- more -->

然后升级了下ruby：

```
$ sudo gem update --system
```
然后升级到了 2.6.14,继续安装：

```
$ gem install cocoapods
```
然而，错误又来了：

```
YAML safe loading is not available. Please upgrade psych to a version that supports safe loading (>= 2.0).
ERROR:  SSL verification error at depth 1: unable to get local issuer certificate (20)
ERROR:  You must add /O=Digital Signature Trust Co./CN=DST Root CA X3 to your local trusted store
ERROR:  While executing gem ... (Errno::EPERM)
    Operation not permitted - /usr/bin/fuzzy_match
```
只好Google了下，然后准备用rvm重新安装ruby

```
$ curl -L get.rvm.io | bash -s stable
```
终端显示：

```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0     51      0  0:00:03  0:00:03 --:--:--    51
100 24090  100 24090    0     0   2939      0  0:00:08  0:00:08 --:--:--  7199
Downloading https://github.com/rvm/rvm/archive/1.29.3.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc
Found PGP signature at: 'https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc',
but no GPG software exists to validate it, skipping.

Installing RVM to /Users/zhouhuiping/.rvm/
    Adding rvm PATH line to /Users/zhouhuiping/.profile /Users/zhouhuiping/.mkshrc /Users/zhouhuiping/.bashrc /Users/zhouhuiping/.zshrc.
    Adding rvm loading line to /Users/zhouhuiping/.profile /Users/zhouhuiping/.bash_profile /Users/zhouhuiping/.zlogin.
Installation of RVM in /Users/zhouhuiping/.rvm/ is almost complete:

  * To start using RVM you need to run `source /Users/zhouhuiping/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
```
然后：

```
$ source ~/.bashrc
$ source ~/.bash_profile
```
查看ruby版本：

```
$ ruby -v
```
列出rvm下所有ruby版本:

```
$ rvm list known
```
安装ruby：

```
$ rvm install 2.4.1
```
时间有点久：

```
Searching for binary rubies, this might take some time.
No binary rubies available for: osx/10.12/x86_64/ruby-2.4.1.
Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
Checking requirements for osx.
Installing requirements for osx.
Updating system...........................................
Installing required packages: autoconf, automake, libtool, coreutils, libyaml, libksba, openssl@1.1.-

.......
Certificates bundle '/usr/local/etc/openssl@1.1/cert.pem' is already up to date.
Requirements installation successful.
Installing Ruby from source to: /Users/zhouhuiping/.rvm/rubies/ruby-2.4.1, this may take a while depending on your cpu(s)...
ruby-2.4.1 - #downloading ruby-2.4.1, this may take a while depending on your connection...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 11.9M  100 11.9M    0     0   121k      0  0:01:41  0:01:41 --:--:--  118k
ruby-2.4.1 - #extracting ruby-2.4.1 to /Users/zhouhuiping/.rvm/src/ruby-2.4.1....
ruby-2.4.1 - #applying patch /Users/zhouhuiping/.rvm/patches/ruby/2.4.1/random_c_using_NR_prefix.patch.
ruby-2.4.1 - #configuring..................................................................
ruby-2.4.1 - #post-configuration.
ruby-2.4.1 - #compiling..............................................................
ruby-2.4.1 - #installing.......
ruby-2.4.1 - #making binaries executable..
ruby-2.4.1 - #downloading rubygems-2.6.14
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  751k  100  751k    0     0  18507      0  0:00:41  0:00:41 --:--:--  7229
No checksum for downloaded archive, recording checksum in user configuration.
ruby-2.4.1 - #extracting rubygems-2.6.14....
ruby-2.4.1 - #removing old rubygems.........
ruby-2.4.1 - #installing rubygems-2.6.14...........................
ruby-2.4.1 - #gemset created /Users/zhouhuiping/.rvm/gems/ruby-2.4.1@global
ruby-2.4.1 - #importing gemset /Users/zhouhuiping/.rvm/gemsets/global.gems.............................|
ruby-2.4.1 - #generating global wrappers........
ruby-2.4.1 - #gemset created /Users/zhouhuiping/.rvm/gems/ruby-2.4.1
ruby-2.4.1 - #importing gemsetfile /Users/zhouhuiping/.rvm/gemsets/default.gems evaluated to empty gem list
ruby-2.4.1 - #generating default wrappers........
ruby-2.4.1 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
Install of ruby-2.4.1 - #complete 
Ruby was built without documentation, to build it run: rvm docs generate-ri
```
安装成功之后就可以安装CocoaPods了：

```
$ sudo gem install cocoapods
```
输入密码，很快安装好了。

```
Password:
Fetching: concurrent-ruby-1.0.5.gem (100%)
Successfully installed concurrent-ruby-1.0.5
Fetching: i18n-0.9.0.gem (100%)
Successfully installed i18n-0.9.0
Fetching: thread_safe-0.3.6.gem (100%)
Successfully installed thread_safe-0.3.6
.
.
.
Installing ri documentation for cocoapods-1.3.1
Done installing documentation for concurrent-ruby, i18n, thread_safe, tzinfo, activesupport, nap, fuzzy_match, cocoapods-core, claide, cocoapods-deintegrate, cocoapods-downloader, cocoapods-plugins, cocoapods-search, cocoapods-stats, netrc, cocoapods-trunk, cocoapods-try, molinillo, CFPropertyList, colored2, nanaimo, xcodeproj, escape, fourflusher, gh_inspector, ruby-macho, cocoapods after 36 seconds
27 gems installed

```
 接着就是 安装一些第三方库，足足等了几个小时。。。  
```
$ cd #项目目录#
zhouhuipingdeMacBook-Pro:JieNiuLife zhouhuiping$ pod install
Setting up CocoaPods master repo
  $ /usr/bin/git clone https://github.com/CocoaPods/Specs.git master --progress
  Cloning into 'master'...
  remote: Counting objects: 1646580, done.        
  remote: Compressing objects: 100% (922/922), done.        
  Receiving objects:  10% (179069/1646580), 35.62 MiB | 27.00 KiB/s 
```



