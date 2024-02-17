---
title: Java学习笔记-基础语法
date: 2017-10-27 15:43:41
categories:
tags:
- Java
- Java基础
keywords:
---


### 1.前言
     
忙了五个多月的项目，代码开发这块总算忙得差不多，真的是劳命伤神啊。
趁这几天空闲下来的功夫，自己也没闲着，准备了解下其他编程语言。
上个礼拜了解了下Python，买了本爬虫的书，2天看完，也照着写了几个简单的爬虫程序。
然后觉得并没什么卵用...

然后，就想了解下后台，首先想到的当然是一直比较流行的编程语言排名第一的Java。于是找了点资料，找到了业界评价不错的马老师的教学视频，也就是《马士兵Java教程》，然后就有了这篇文章。
 
**PS:这篇纯属基础笔记，没啥技术含量。**
    
<!-- more -->    

### 2.正文——笔记

话不多少，直接上笔记吧

>**1.标识符**

* Java对各种变量、方法和类等要素命名时使用的字符序列称为标识符。
    * 凡是自己可以起名字的地方都叫标识符，都遵守表示符的规则。
* Java标识符命名规则：
     * 标识符由字母、下划线“_”、美元符"$"或数字组成。
     * 标识符应以字母、下划线、美元符开头。
     * Java标识符大小写敏感，长度无限制。
* 约定俗成：Java标识符选取因注意“见名知意”且不能与Java语言关键字重名。
    
合法的标识符 | 不合法的标识符
--------- | ---------
HelloWorld  | class
DataClass   | DataClass#
_983    | 98.3
$bS5_c7  |  Hello World



>**2.关键字**

* Java中的一些赋以特定的含义，用做专门用途的字符串称为关键字（keyword）。
    * 大多数编辑器会将关键字用特殊方式标出。
* 所有Java关键字都是**小写英文**。
* **goto**和**const**虽然从未使用，但也被作为Java关键字保留。

第一列 | 第二列 | 第三列 | 第四列 | 第五列            
--------- | --------- | --------- | --------- | -------------
abstract | default | if | private | this 
boolean | do | implements | protected | throw 
break | double | import | public | throws
byte | else | instanceof | return | transient
case | extends | int | short |  try
catch | final | interface | static | void 
char | finally | long | strictfp | volatile
class | float | native | super | while
const | for | new | switch | null 
continue | goto | package | synchronized | ...



>**3.Java常量**

* Java的常量值用字符串表示，区分为不同的数据类型。
    * 如整型常量 123
    * 实型常量 3.14
    * 字符串常量 ‘a’
    * 逻辑常量 true、false
    * 字符串常量“helloworld”
*注意：区分字符常量和字符串常量
*注意：“常量”这个名词还会用在另外其他语境中表示值不可变的变量：
        * 参见final关键字
   
   
>**4.Java变量**

* Java变量是程序中最基本的存储单元，其要素包括变量名，变量类型和作用域。
* Java程序中的每一个变量都属于特定的数据类型，在使用前必须对其声明，声明格式为：
    * type varName [=value][{,varName[=value]}]
* 例如:

```java
                       int i = 100;
                       float f = 12.4f;
                       double d1,d2,d3 = 0.123;
                       String s = "hello";
```

     
   
* **从本质上讲，变量其实是内存中的一小块区域，使用变量名来访问这块区域，因此，每一个变量使用前必须要先申请（声明），然后必须进行赋值（填充内容），才能使用。**

* 程序执行过程：

![](/图片测试/幻灯片22.JPG)
(code segment:代码区；data segment:数据区；stack:栈区；heap:堆区。)


>**5.Java基本数据类型**

* Java中定义了4类8种基本数据类型。
    * 逻辑型——boolean
    * 文本型——char
    * 整数型——byte,short,int,long
    * 浮点整型——float,double

* 1.逻辑型 **Boolean**
    * boolean类型适用于逻辑运算，一般用于程序流程控制。
    * boolean类型数据只允许取值true或false，不可以0或者非0的整数代替true和false，这点和C语言不同。
    * 用法举例：
    
    ```java
        boolean flag;
        flag = true;
        if(flag){
            //do something
        }
    ```
    
* 2.字符型 **char**
    * char型数据用来表示通常意义上“字符”
    * 字符常量为用单引号括起来的单个字符，例如：
    ```java
                char eChar = 'a';
                char cChar = '中';
    ```
    * Java字符采用**Unicode**编码，每个字符占用两个字节，因而可用十六进制编码形式表示，例如：```java
     char c2 = '\n';
    ```
    * 补充：2进制、10进制、16进制之间的转换
        * 1101 —— 1 * 1 + 0 * 2 + 1 * 4 + 1 * 8
        * 13 —— 1 + 4 + 8 —— 1101
        * 1101 —— D
* 3.整数类型 **byte、short、int、long**
    * Java各整数类型有固定的表数范围和字段长度，其不收具体操作系统的影响，以保证Java程序的可移植性。
    * Java语言整型常量的三种表示形式：
        * 十进制整数，如：12，-314，0。
        * 八进制整数，要求以0开头，如：012。
        * 十六进制数，要求0x或0X开头，如：0x12。
    * Java语言的整型常量默认为int型，声明long型常量可以后加“l”或者“L”，如：```java
        int i1 = 600; //正确  long l1 = 8888888888L;//正确，必须加l或者L，否则会出错
    ```
类型  |  占用存储空间  | 表数范围
--------- | --------- | ---------
byte | 1字节 | -128 ~ 127
short | 2字节 | -2^15 ~ 2^15 - 1
int | 4字节 | -2^31 ~ 2^31 - 1
long | 8字节 | -2^63 ~ 2^63 - 1

* **4.浮点类型**
    *   与整数类型类似，Java浮点类型有固定的表数范围和字段长度，不受平台影响；
    *   Java浮点类型常量有两种表示形式：
        *   十进制数形式，例如：3.14  314.0   .314
        *   科学计数方式，例如：3.14e2    3.14E2  100E-2
    *   Java浮点型常量默认认为double类型，如要声明一个常量float型，则需在数字后面加f或F，如：
        * double d = 1234.9;
        * float f = 12.4f;  //必须加“f”否则会出错
    *   下面列出Java的各种浮点类型：
    
类型 | 占用存储空间 | 表数范围
--------- | --------- | ---------
float | 4字节 | -3.403E38 ~ 3.403E38
double | 8字节 | -1.798E308 ~ 1.798E308


* **5.基本数据类型转换**
    * boolean 类型不可以转换为其他的数据类型。
    * 整型，字符型，浮点型的数据在混合运算中的相互转换，转换时遵循以下原则：
        * 容量小的类型自动转换为容量大的数据类型；数据类型按容量大小排序为：```Java
byte,short,char->int->long->float->double
bute,short,char之间不会相互转换，他们、三者在计算时首先会转换为int
```
    * 容量大的数据类型转换为容量小的数据类型时，要加上强制转换符，但可能造成精度降低或溢出；使用时要格外注意。
    * 有多种类型的数据混合运算时，系统首先自动的将所有数据转换成容量最大的那一种数据类型，然后在进行计算。
    * 实数常量（如：1.2）默认为 double。
    * 整数常量（如：123）默认为int。
    ```java
    public class TestCovert {
   
   	  public static void main(String[] args) {
   
           	int i = 1,j;
           	float f1 = 0.1f;
           	float f2 = 123;
           	long l1 = 12345678,l2 = 88888888L;
           	double d1 = 2e20,d2 = 124;
           	byte b1 = 1,b2 = 2,b3 = 127;
           	j = 1;
           	j = j + 10;
           	i = i / 10;
           	i = (int)(i * 0.1);
           	char c1 = 'a',c2 = 125;
           	byte b = (byte)(b1 - b2);
           	char c = (char)(c1 + c2 - 1);
           	float f3 = f1 + f2;
           	float f4 = (float)(f1 + f2*0.1);
           	double d = d1 * i + j;
           	float f = (float)(d1*5 + d2);
              
           	System.out.println("c1 is " + c1);
           	System.out.println("b is " + b);
           	System.out.println("c is " + c);
           	System.out.println("f3 is " + f3);
           	System.out.println("f4 is " + f4);
           	System.out.println("d is " + d);
           	System.out.println("f is " + f);
   
   	 }
          
    }
    ```
   用终端编译该文件

   ```java
    $ javac TestCovert.java
   ```
    
   编译器报错：

   ![](/图片测试/TestConvertError.jpg)
   
   参考写法：
```Java
public class TestCovert {
    
    	public static void main(String[] args) {
    
    		int i = 1,j;
    		float f1 = 0.1f;
    		float f2 = 123;
    		long l1 = 12345678,l2 = 88888888L;
    		double d1 = 2e20,d2 = 124;
    		byte b1 = 1,b2 = 2,b3 = 127;
    		j = 1;
    		j = j + 10;
    		i = i / 10;
    		i = (int)(i * 0.1);
    		char c1 = 'a',c2 = 125;
    		byte b = (byte)(b1 - b2);
    		char c = (char)(c1 + c2 - 1);
    		float f3 = f1 + f2;
    		float f4 = (float)(f1 + f2*0.1);
    		double d = d1 * i + j;
    		float f = (float)(d1*5 + d2);
    
    		System.out.println("c1 is " + c1);
    		System.out.println("b is " + b);
    		System.out.println("c is " + c);
    		System.out.println("f3 is " + f3);
    		System.out.println("f4 is " + f4);
    		System.out.println("d is " + d);
    		System.out.println("f is " + f);
    
    	}
          
}
    ```


> **6.程序格式**

* 大括号对齐；
* 遇到{缩进，Tab/shift + Tab;
* 程序块之间加空行；
* 并排语句之间加空格；
* 运算符两侧加空格；
* }前面有空格；
* 成对编程。

> **7.最后补充两个练习题**

* 输出1~100内前5个可以被3整除的数。
* 输出101~200内的质数。

>// 输出1~100内前5个可以被3整除的数
>参考写法

```java
public class TestBreak {

	public static void main(String[] args) {

		int num = 0;
		for(int i = 1;i < 100;i ++){
			if(i%3 == 0) {
				System.out.println("get one : " + i);
				num ++;
				if (num ==  5) {
					break;
				}
			}
		}
	}
}
```
>//输出101~200内的质数
>参考写法

```Java
public class TestPrime {

	public static void main(String[] args) {

		for(int i = 101;i < 200;i += 2) {

			for(int j = 2;j < i;j ++){
				if (i%j == 0) {
					break;
				}else if (j == (i - 1)) {
					System.out.println(i);
				}
			}
		}
	}
}
```


