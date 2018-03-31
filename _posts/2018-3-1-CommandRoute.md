---
layout:     post
title:      "mobileYY_CommandRoute"
subtitle:   "手Y命令号路由系统"
date:       2018-3-1 00:00:00
author:     "DAWNOTDOWN"
header-img: "img/post-bg-CommandRoute.jpg"
tags:
    - 原创
    - iOS开发
---
>"整理了一下以前参与的命令号路由这一块儿，这一块儿基本是推送落地的一个基石，现在越来越多的业务需要命令号路由系统，基本上每有一个业务都需要新加一个命令了~"  

# URL
URL，Uniform Resource Locator，统一资源定位符，是一种 URI，用定位的方式来精确描述。  
URN，Uniform Resource Name，统一资源名称，也是一种 URI，用名字来命名但不指定如何定位某个Resource。  
URI，Uniform Resource Identifier，统一资源标识符。
URN如同一个人的名称，而 URL代表一个人的住址。换言之，URN 定义某事物的身份，而 URL 提供查找该事物的方法。

# URL Scheme  
在https://dawnotdown.github.io这串网址里，scheme就是https，标识这个URL最初始的位置。  
在iOS里URL Scheme一般用于苹果应用间的跳转。我们在一个应用的plist配置里定义这样一个scheme来告诉苹果系统这个应用的标识，通过[UIApplication sharedApplication] openURL或者Safari等方式来实现跳转的过程，对应的App通过application:openURL:...等方法来响应这一过程，进而可以实现某些App间的需求。 

# App-Inside Command
对一个用户来说，每天完成App间的相互跳转的频率可能比在一个App内的跳转的频率还要低。对于开发来说更是，业务或者模块间相互的跳转只会越来越多，伴随着可能就是越来越大的耦合。  
App-Inside Command，App内的命令号希望可以通过一串像命令一样的东西，来完成App内的各种跳转过程。  
**如何跳转**   
从Window层开始找到当前的VC或Nav的某一根视图或TabBarC下的某一根视图  
在当前VC进行present或dismiss掉Nav的视图然后再push   
**什么是命令号**  
以前手Y的命令号是不规范的URL格式，  
比如：  
yymobile://Channel/Live/83191303/83191303，同一业务的path会变    
yymobile://Channel/Live/45294452/452294452?tpl=16777217，同一业务的path和query都会变    
我们希望把它统一改进为URL的样子并且支持路由功能。
比如：
yymobile://Channel/Live?sid=45294452&ssid=452294452&tpl=16777217，同一业务只有query会变


# App-Inside URL
对于上面的命令号我们改进为了下面的标准URL格式
![图片1](/img/mobileYY-CommandRoute_yymobile.jpg)
并且对原来旧的提供了支持，重新设计了路由，自此命令号升级为了路由系统。  
### 路由的作用
1.解决了模块间页面切换和跳转的难题，解除了这种场景下的耦合性  
2.便于服务器下发动态配置文件来配置某些场景和活动的跳转逻辑  
3.统一了iOSAndroid两端的页面跳转逻辑，甚至能统一三端的请求资源的方式  
4.解决了外部跳转到App内部一个界面的问题，比如推送,3D Touch等外部请求  
5.改变既定跳转线路，达到热修复的目的  
### 路由的实现
**我们的考虑**  
我们在实现路由和沿用原命令号的方式主要希望达到以下几点：  
1.各业务方可以自己维护跳转代码  
2.路由过程对业务方透明  
3.兼容原命令号  

**设计流程**
![图片2](/img/mobileYY-CommandRoute.png)
也可点这里→[图](https://www.processon.com/view/link/5ab1380be4b007d2510ceec9)  

大概流程如下：   
1.命令号格式由各业务维护，包括命令号的执行逻辑，比如跳转。  
我们通过YYURINavigationCenter+XXXXClass的分类方式的设计由各业务方自己添加跳转逻辑。  
每个分类下可注册多个命令号格式，每个命令号格式需实现两个方法:注册命令号到路由里
和定义自己的跳转逻辑  
并且我们通过带名传参的宏简化了业务注册的代码，如下图  
![图片3](/img/mobileYY-CommandRoute_code2.jpg)
各业务方只需要实现如下图的简单代码就可以完成一个命令号格式
![图片4](/img/mobileYY-CommandRoute_code1.jpg)  
2.App启动去注册路由动作  
App启动的时候去调用所有YYURINavigationCenter类的以_UriNavigationCenterResign开头的方法（通过宏定义创建），以完成在YYRouteManager里YYRouteHandlerBlock与命令号的对应关系注册    
3.业务方调用YYRouteManager的单例来传入一个命令号  
传入命令号以及VC和动画参数等，打包这些参数给YYRouteManager，YYRouteManager内部根据此命令号创建路由对象YYRoute，遍历已注册的所有路由关系，创建或使用已创建的命令号格式匹配对象YYRouteMatcher，每一种命令号格式对应一个YYRouteMatcher    
4.YYRouteMatcher的匹配操作  
遍历所有YYRouteMatcher，会先判断传入的命令号是否是标准URL  
如果不是标准URL，通过YYRouteMatcher对应的命令号格式生成的匹配正则表达式匹配此命令号。
如果是标准URL，则通过URL规则去匹配命令号，为了兼容旧命令号，如果这里匹配不通过，会当做不标准URL继续尝试匹配。
直到有YYRouteMatcher可以匹配当前的命令号  
5.执行该命令号格式对应的block方法


