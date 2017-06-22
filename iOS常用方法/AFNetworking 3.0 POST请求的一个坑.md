---
layout: post
title:  "AFNetworking 3.0 POST请求的一个坑"
categories: iOS 开发
toc: true
tags: [AFNetworking  Objective-C]
date: 2016-12-19 20:46:25
---

最近做一个项目，用到了网络通信。为了稳定和缩短时间，果断选择了使用第三方框架AFNetworking. 本以为稍微对AFN封装一下就万事大吉了，可是没想到，自己调入了一个大坑！ 下面详细介绍下从问题的由来到入坑再到解决的过程： 

### 1.项目需求
项目中有一个业务是登录功能，需要实现移动端通过POST请求向后台发送数据，然后从响应头里把Set-Cookie剥离出来缓存到本地。

### 2.解决思路
项目需求已经很明确了，后台也将接口暴露给了我，接下来我只需要封装一个函数用于和后台服务器进行交互就可以了。于是：

````
*!

*使用密码登录，并实现成功和失败的回调

*

*@param tel用户ID

*@param passwordMD5密码MD5

*@param success成功回调

*@param fail失败回调

*/

+ (void) loginUser:(NSString*)tel andPassword:(NSString*)passwordMD5 success:(SuccessMoreBlock)success fail:(FailBlock)fail;
````
就这样写了。用这个方法封装了AFN。在理想情况下，应该是每一次我调用，如果用户名和密码正确的话服务器会给一个响应头，里面有Set-Cookie的属性。

### 3.问题来了——顺利入坑
终于，问题来找我了。这个函数本身并没有什么问题，但是实际用起来会发现一个奇怪的现象：
 用第三方工具测试 证明 ，服务器的response里面 每次都有Set-Cookie  ,但是使用AFN在程序已启动时可以获取到Set-Cookie ，获取后 间隔几分钟再次获取就获取不到了 。要是程序重启，依旧第一遍能获取到。
这是第一次POST：

![Paste_Image.png](/images/photo4.png)
这是第二次POST:

![Paste_Image.png](/images/photo5.png)
 
### 4.跳出大坑
坐下来仔细思考发现，服务器用的 Tomcat, 后台是 Java 写的，这个 Set-Cookie 是服务器要求客户端保存 cookie 的命令. Set-Cookie 后面跟的内容就是需要保存的内容.可以看一下,服务器返回的是 Set-Cookie:JSESSINID=XXXXX这个 JSESSIONID 是 sessionid, 不同的后台 sessionid 的名字不一样,  但作用都是一样的,是用来标记客户端的.HTTP 链接是不能保持状态的,那么服务器怎么知道某个客户端是否与服务器交互过呢?就要靠这个 sessionid,服务器每次发现一个新的客户端,就会生成一个 session 同时把 sessionid 放到 cookie 里面下发给客户端.客户端接收到 sessionid 后必须保存下来,每个 requestheader 都要发送这个 sessionid, 服务器根据 sessionid 检索出相应的 session, 就可以根据 session 获取客户端的某些状态,比如是否登录之类的。

但是为什么使用AFN会出这样的问题呢？而且是第一次POST都正常，再以后POST就不返回Cookie了，然而程序重启之后再POST又正常了？说明服务器肯定没有问题，是自身的问题。

于是用第三方工具测试发现，单独只传用户名和密码可以收到Set-Cookie,我如果在请求头里再添加一个Cookie参数，就收不到Set-Cookie了。

于是推测，使用AFN POST的时候第一次是没有获取到Set-Cookie,所以POST的时候会收到服务器返回的Set-Cookie,第二次再POST的时候带上了这Cookie。

但是，我封装的这个函数每次都是重新建一个AFN对象，参数都初始化了，为什么会带上Cookie呢？于是手动设置AFN的响应头，把Cookie参数设为空，再次测试，发现每次POST都会收到服务器的Set-Cookie参数！

分析，AFN在内部可能做了优化，当POST一次的时候它会去获取Cookie，然后自己就缓存起来了，当下一次再次POST的时候，如果它发现POST的URL有缓存过Cookie它就默认带上了已缓存好的Cookie。

### 5.简单的吐下槽
感觉AFN这么做也合理也不合理，网络操作流程应该是这样子，但是在用户没有主动设置的情况下就默认带上Cookie也不说明就是AFN的不对了......













