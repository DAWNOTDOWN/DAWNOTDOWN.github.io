---
layout:     post
title:      "mobileYY_Channel-Modularization"
subtitle:   "手Y直播间组件化"
date:       2018-1-29 00:00:00
author:     "DAWNOTDOWN"
header-img: "img/post-bg-mobileYY-Channel-Modularization.jpg"
tags:
    - 原创
    - iOS开发
---
>"来YY快1年了,也做了很多直播间内的业务，今天介绍一下我们正在优化的手Y直播间的组件化加载"  

# 手Y的直播间  
手Y的直播间本质上就是YY的频道，进直播间等同于先进了某个频道，对开播侧的主播来说也是如此，然后再实现频道内的几乎所有功能，频道是直播间的一个剥离具体业务的抽象。 
 
**手Y直播间需要有PC频道物理相关的所有功能:**  
1.频道管理功能  
管理员有把用户拖进或踢出频道的功能，也有把用户拖上麦序或拖离麦序的功能  
2.频道号  
常见的频道号有显式的官频号,短位和长位号以及子频道号  
开发层面的频道号只识别顶级频道号TopSid和子频道号SubSid的对应关系  
3.视频流与直播间  
视频流不依赖于直播间，可能正在进频道的过程时就已经开始走视频流拉取流程。比如开播端  
4.直播间主播和首麦  
直播间的主播指的就是当前在首麦的用户，它不一定是该直播间的拥有者，也不一定是当前直播间的视频流上传者，麦序是麦序，视频流是视频流。比如YY交友业务里的乱斗玩法，首麦和次麦两路流都上传，观众侧看到的可能是次麦  
5.进频道与频道切换  
​进频道是需要登录的，没登录账号就能进直播间是因为使用了匿名账号登录，当登录账号或者切换账号时，需要退出直播间然后重新进一次直播间。当然，在一个频道里切换子频道也是需要这样操作  
...  
​等等  
​    
**手Y直播间也需要有频道里业务相关的所有功能：**  
1.公屏  
2.弹幕  
3.麦序  
4.送礼  
5.礼物流水  
6.各种h5玩法（周星周卡,幸运转盘...）  
7.私聊  
8.清晰度切换    
9.连麦  
10.关注  
...  
**还有结合手机业务相关的所有功能：**  
1.转屏  
2.音频模式  
3.清屏  
4.分享  
5.大礼物  
6.贵族  
7.玩法（主要在主播端，欢乐篮球,欢乐斗,还有最近做的吃货脸萌,魔法手势,变声...）   
8.美颜（主要在主播端）    
9.贴纸（主要在主播端）     
10.音乐（主要在主播端）     
11.贡献榜  
12.h5活动条    
...  
​等等，大概有5 60种...
​
​  
# 直播间的组件化
手Y的直播间按使用人群分可分为两类，观看端和开播端；按模式分可分为至少三类，麦序模式，主席模式，自由模式；按模板分可分为开播端模板，娱乐模板，游戏模板，抓娃娃模板等等。不同的端，不同的模式，不同的模板，会加载不同的业务；不同的业务，拥有不同的view，各个view之间有业务上的交互，还有view之间的约束依赖。  
现有业务在小小的屏幕上占一个坑，以后的业务又还想再进来。旧的模板满足不了日益增多的功能，新的模板又需要兼容一部分旧的模板。  
基于模板和模式的手Y直播间加载各个业务view，采用了组件化的方式，重点考虑了以下几个要素：      
组件独立处理业务，不耦合太严重。  
组件之间需要交互，但要低耦合。  
组件应该在模板下有某种方式自动加载。    
组件的约束应该可以方便的设置和修改。  
不同的模板要允许自定义加载不同的组件。  
  
所以，我们重新设计了直播间的组件化加载方式。  

### 一个组件
一个组件至少需要以下部分：  
1.组件的View及其业务逻辑  
2.组件的布局  
3.组件的加载  
代码上由以下构成：  
1.ModuleView:YYChannelModuleView，是被添加到直播间的一个组件的view的入口，这个view就是这个组件在直播间的containerView  
2.AreaView：ModuleView的parentView，建立与直播间superView的约束，ModuleView平铺在这个View上  
3.ViewModel:YYChannelModuleUIManager，每个YYChannelModuleView都关联一个UIManager的ViewModel，是组件的业务逻辑所在，相当于moduleView的ViewController  
4.DataProvider:YYChannelModuleDataProvider，为moduleView提供网络请求和数据  
5.IXXXModuleBridge,ModuleBridge:YYBridge <IXXXModuleBridge>与ModuleImplementation:YYImplementation <IXXXModuleBridge>，为其他组件暴露接口，方便其他组件调用本组件的逻辑。  
7.ModuleStatistic:NSObject <IModuleStatistic>，数据上报使用  
8.Views，是组件所用到的自定义SubViews

### 组件加载
这里拿观众端娱乐模板+麦序模式举起一个栗子
![图片1](/img/mobileYY-Channel-Modularization.png)
也可点这里→[图](https://www.processon.com/view/link/5aafe49be4b0e935339228b5) 
 
大概流程如下：  
1.娱乐模板YYEChannelTemplateController的super YYBaseChannelViewController    
2.基础模板YYBaseChannelViewController刚加载的时候创建ModeController    
3.ModeController创建ModeLoader  
4.modeLoader根据麦序模式创建ModeModuleFactory和ModeLayoutFactory
ModuleFactory中创建YYModuleLoader，设置moduleConstructBlock，并用moduleId注册到一个map中，在后续YYChannelModeLoader加载某个组件时，通过moduleId取出moduleConstructBlock并执行，从而初始化并添加moduleView。
布局的管理由LayoutFactory管理，对每个组件创建YYLayoutLoader，包含p_layoutBlock(竖屏约束)，l_layoutBlock（横屏约束）和fixed_layoutBlock（固定约束），并同样的以moduleId作为key注册到一个map中，方便后续YYChannelModeLoader加载时使用  
5.执行loadModuleWithModuleId开始加载组件  
6.首先加载完AreaLayout（拿moduleLoader.areaConstruct()用layoutLoader根据moduleId选择加约束在YYBaseChannelViewController的view还是modalView或是contentView）  
7.然后加载Module（ModuleLoader调createModule通过moduleViewConstruct()生成一个YYmodule，然后area.areaView addSubView:module.moduleView并平铺）
8.调用p_moduleDidLoad方法，使用module.moduleView设置业务逻辑  
9.YYChannelModeLoader继续加载，遍历剩下的defaultModuleList，并加载每个moduleId，根据moduleId，从ModuleFactory中取出moduleConstructBlock，从LayoutFactory中取出layoutBlock，从areaFactory中取出areaConstruct，由此构造出完整的组件并添加到直播间view中。


### 层级关系
上图已有，拿出来再说一下
![图片2](/img/mobileYY-Channel-Modularization_hierarchy.jpg)

ViewController的有三个view，分别是ChannelVC的self.view，self.contentView和self.modalView。默认情况下，组件是加载到contentView中，modalView一般用来显示直播间的弹框等模态的view，而且一些组件比如onlineListModuleView，因为他覆盖在整个直播间上，相当于push了一个新的vc，所以也加载了modalView中。 


>组件化使得各业务可以独立处理，相互之间耦合程度大大降低。各业务View在moduleLoader和layoutLoader的配合下加载，可以方便的设置,修改和新增。各业务逻辑集中在ModuleView和ViewModel内部，如果业务组件间需要交互，通过抛通知的方式或者GetCore注册调用或单例的方式拿
