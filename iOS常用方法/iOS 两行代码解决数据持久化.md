---
layout: post
title:  "iOS 两行代码解决数据持久化"
categories: iOS 开发
toc: true
tags: [数据持久化]
date: 2016-02-18 20:46:25
---


### 前言
在实际的iOS开发中，有些时候涉及到将程序的状态保存下来，以便下一次恢复，或者是记录用户的一些喜好和用户的登录信息等等。 这就需要涉及到数据的持久化了，所谓数据持久化就是数据的本地保存，将数据从内存中迁入到存储器上。网上有很多种数据持久化的方法，如实现自己实现I/O、数据库、云或则走第三方接口等等。但是有时候可能只是进行一些简单的数据存储，如用户的偏好设置、用户的sessionID等等，这时候使用上述方法便显得有点兴师动众了，现在需要一种更加轻量化的操作方式。
 
### 一、认识 NSUserDefaults
为了寻求上述问题的解决方案，查阅apple官方文档发现，有一个类NSUserDefaults是苹果设计专门用来解决这个问题的：
````
  NSUserDefaults is a hierarchical persistent interprocess
 (optionally distributed) key-value store, optimized for storing user settings.
````
翻译大致如下：
````
NSUserDefaults 是一种进程间（任意分布）的分层级持久化键-值存储，为存储用户设置而优化。
````
详细说明可以查阅官方文档，这里只介绍其使用。
现在，我们已经找到了一种轻量级的数据持久化解决方案了，为什么说它轻量级呢。因为apple官方设计它的目的就是为了解决用户设置的存储问题，下面就来介绍它的使用。
### 二、使用 NSUserDefaults
由于NSUserDefaults是一种进程间的解决方案，所以我们可以在任意一个进程中调用它来访问和存储用户的信息。
举个例子：我们要对用户的用户名进行数据的持久化操作
````
NSUserDefaults *userDefaults = [NSUserDefaults standardUserDefaults];
[userDefaults setObject:@"whf" forKey:@"name"];
````
通过上面这两行代码，我们就已经将用户的姓名通过键值对的方式存储到本地了。不需要指定数据的存储位置，一切由系统搞定，我们只需要告诉系统我们要存什么。如果多次存储的是同一个键的值，那么这个键的值是根据最后一次的值定的，也就是说系统是覆盖写，而不是追加写最后返回的是数组。

接下来演示取数据的过程：在任意线程中，我们调用
````
NSUserDefaults *userDefaults = [NSUserDefaults standardUserDefaults];
NSString *userName = [userDefaults objectForKey:@"name"];
````

这两句就可以从存储器上获得我们要的数据了，如果数据不存在，那么返回的对象就是nil。

### 三、底层实现机制
通过NSUserDefaults的使用，发现程序重新运行数据依旧存在，那么这个数据肯定是被存储在了手机的存储器上。现在来探寻它的实现机制：
````
NSUserDefaults *userDefaults = [NSUserDefaults standardUserDefaults];
    [userDefaults setObject:@"123" forKey:@"name"];
    NSString *userName = [userDefaults objectForKey:@"name"];
    
    NSLog(@"%@",userName);
    NSString *homeDirectory = NSHomeDirectory();
    NSLog(@"homeDire --------%@",homeDirectory);
````

运行结果：
![Snip20160807_8.png](/images/photo6.png)
根据路径进入沙盒发现，在沙盒的Library/Preferences/目录下发现多出了一个com.itripbuyer.Date-Persistence.plist的plist文件。

![Snip20160807_9.png](/images/photo7.png)
打开后发现里面有一个键值对，并且就是我们刚刚操作的数据。于是我猜测，通过我们刚才的两行代码，系统将我们的数据转换成了一个plist文件，这个文件中装载的是一些键值对。

### 四、灵活巧用
NSUserDefaults 官方给出的用途是存储用户的Setting，但是通过上述操作发现，程序中凡是涉及到键值对的存储，都可以使用NSUserDefaults来实现，即使不是键值对的形式，转换成键值对也要用NSUserDefaults来实现，这样既省时又省力，还能用最简洁的代码换来最稳定的数据持久化操作。

###### 欢迎大家关注喜欢并留言指正批评。














