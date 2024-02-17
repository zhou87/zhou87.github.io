---
title: 测试下Hexo
comments: true
date: 2017-04-17 16:06:01
tags:
    - hexo博客搭建
    

---
#你好
 这是只是一个测试。。。

I like [Google](https://www.baidu.com/)
<br>
<br>

<!-- more -->
>段落

这是一个段落
=============
这是第二个段落
-------------
>标题

<br>
##这也是一个段落H4##

#### 这又是一个段落H5
#### 

>引用
> 这是一个引用

<br>

>列表

<br>
* 可以使用‘*’作为标记
+ 也可以使用'+'
- 或者‘-’

1. 第一层
  + 1-1
  + 1-2
2. 第二层
   1. 2-1
   2. 2-2

05\. 可以使用 ： 数字\. 来取消显示为列表

>代码 <br>

<html>     
      
   //block的实现方法(向表中添加一调信息，添加成功后将这调信息insert到tableView的第一个单元格)
    __weak NSMutableArray *weakarray = _arr;
    __weak UITableView *weaktable = table;
    __weak UIActivityIndicatorView *weakActivity = Activity;
    AddVc = [[AddViewController alloc] init];
    AddVc.block = ^(Patient *patient) {
        //开始旋转
        __strong UIActivityIndicatorView *strongActivity = weakActivity;
        [strongActivity startAnimating];
        NSLog(@"patient:%@",patient.leftOne);
        //向数据库中添加一条信息
        int result =  [DBManger inserPatient:patient table:PatientsSqlite];
        __strong NSMutableArray *strongarray = weakarray;
        __strong UITableView *strongtable = weaktable;
        NSLog(@"strongarray:%ld",strongarray.count);
        if (result) {
            //停止旋转
            [strongActivity stopAnimating];
            //向table中添加一个cell
            NSIndexPath *refreshCell = [NSIndexPath indexPathForRow:0 inSection:0];
            NSArray *insertIndeaPath = [NSArray arrayWithObjects:refreshCell, nil];
            [strongarray insertObject:patient atIndex:0];
            NSLog(@"%lu",(unsigned long)strongarray.count);
            [strongtable insertRowsAtIndexPaths:insertIndeaPath withRowAnimation:UITableViewRowAnimationAutomatic];
            
        }
    };


</html>    

>分割线 <br>

* **
---- ---
____ ____

>超链接 <br>

[百度](https://www.baidu.com/  "Baidu")

[icon.png](./images/icon.png)

<http://www.baidu.com>
<123@foxmail.com>

>图像 <br>

![github](http://img4.imgtn.bdimg.com/it/u=783479549,2303394359&fm=23&gp=0.jpg "Github")

>强调 <br>

这是用 *斜体* 来演示 的_斜体_

**加粗**   __加粗__

>转义  <br>

这是用来 \*斜体 \*   的 \_ 斜体 \_

>删除线 <br>

~~删除线~~

>代码块  <br>

```JS
 //block的实现方法(向表中添加一调信息，添加成功后将这调信息insert到tableView的第一个单元格)
    __weak NSMutableArray *weakarray = _arr;
    __weak UITableView *weaktable = table;
    __weak UIActivityIndicatorView *weakActivity = Activity;
    AddVc = [[AddViewController alloc] init];
    AddVc.block = ^(Patient *patient) {
        //开始旋转
        __strong UIActivityIndicatorView *strongActivity = weakActivity;
        [strongActivity startAnimating];
        NSLog(@"patient:%@",patient.leftOne);
        //向数据库中添加一条信息
        int result =  [DBManger inserPatient:patient table:PatientsSqlite];
        __strong NSMutableArray *strongarray = weakarray;
        __strong UITableView *strongtable = weaktable;
        NSLog(@"strongarray:%ld",strongarray.count);
        if (result) {
            //停止旋转
            [strongActivity stopAnimating];
            //向table中添加一个cell
            NSIndexPath *refreshCell = [NSIndexPath indexPathForRow:0 inSection:0];
            NSArray *insertIndeaPath = [NSArray arrayWithObjects:refreshCell, nil];
            [strongarray insertObject:patient atIndex:0];
            NSLog(@"%lu",(unsigned long)strongarray.count);
            [strongtable insertRowsAtIndexPaths:insertIndeaPath withRowAnimation:UITableViewRowAnimationAutomatic];
            
        }
    };
 

```
>表格 <br>

name | age | sex | height | hight
---- | ---
LearnShare | 12 | meal | 120 | 173
Mike | 34 | femeal | 140 | 189




