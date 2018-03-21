---
layout:     post
title:      "mobileYY_Channel-Templatization"
subtitle:   "手Y直播间模板化"
date:       2018-2-1 00:00:00
author:     "DAWNOTDOWN"
header-img: "img/post-bg-mobileYY-Channel-Templatization.jpg"
tags:
    - 原创
    - iOS开发
---
>"PC YY直播间有很多模板，娱乐模板，游戏直播模板，教育模板，交友模板，乱斗模板等等，不同的模板有不同的业务，负责不同的功能，手Y的直播间模板化做法也是由此而生"  

# 直播间的模板化

模板自PC YY已有，直播间的模板化本质是根据不同的模板ID，事先定义该模板具有的业务，然后在进频道或者切换频道时加载这个模板，即这些模板具有的业务。 

**模板化的必要性**   
1.受制于不同的业务分类，一个频道可能有多种不同的业务加载和显示  
2.大部分业务是基于开播业务的，这就要求观众在进频道后就需要知道当前主播的开播类型，低成本的做法是主播一开播就告诉后台一些开播种类相关的参数，观众在进这个频道时拿到这些参数，根据这些参数来加载与当前开播方式相关的业务，而不需要每个观众在进频道后都收到不同的业务后台的不同的广播来告诉加载什么业务  
3.减少了代码冗余和低效率的使用，大部分情况下A模板是不需要B模板的业务的  
4.降低了潜在的耦合
 
**手Y直播间模板化**   
手Y直播间的模板化参照PC YY的模板化的思想，把业务组件独立，在不同的模板里加载  

​
​  
# 观众端的模板化
这里拿观众端娱乐模板举起一个栗子
![图片1](/img/mobileYY-Channel-Templatization_audience.png)
也可点这里→[图](https://www.processon.com/view/link/5aaf7191e4b0d248ffd08dcc) 

大概流程如下：  
1.根据从外源比如首页拿到的的豆腐块数据拿到topSid,subSid,tpl（PC模板号）,uid,isGame等等信息，构造YYChannelUserInfo     
2.传入presentChannelViewControllerWithUserInfo:方法  
3.创建YYParentChannelViewController父类，并用UINavigationController的子类包裹    
4.查找模板类型    
5.告诉SDK开始进频道    
6.加载指定的模板VC   
7.加载相应的组件  

# 开播端的模板化
开播端相对稍微复杂一点，这里拿普通开播模板举起一个栗子
![图片2](/img/mobileYY-Channel-Templatization_live.png)
也可点这里→[图](https://www.processon.com/view/link/5ab0cf8ce4b0e9353394a197) 

开播端会先加载一个可上传标题，封面等的一个预览页，这时候主播已经能看到自己了，点击开播按钮后才正式进入直播间，加载模板  
<br>
大概流程如下：  
1.从手机开播方式获取到当前的开播模式（普通，轮麦，M直播）    
2.传入presentLiveParentViewController:方法  
3.调p_presentNormalLiveChannelViewController方法    
4.创建LiveParentViewController父类    
5.记录当前的模板类型    
6.用UINavigationController的子类包裹LiveParentViewController   
7.presentViewController后viewDidLoad  
8.加载主播音视频层界面
9.添加预览界面  
10.点击开播按钮，开始3s倒计时动画界面  
11.移除预览界面  
12.创建当前的模板类LiveChannelViewController，alpha=0，添加在LiveParentViewController上  
13.同步加载所需的所有组件  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在3s结束后，LiveChannelViewController，alpha=1

