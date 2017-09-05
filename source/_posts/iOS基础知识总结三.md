---
title: iOS基础知识总结三
tags:
  - 笔记
category:
  - iOS开发
date: 2017-09-04 20:00:33
---

# viewDidLoad、viewWillDisappear、viewWillAppear等方法的执行顺序和作用
* viewWillAppear：视图即将可见时调用。默认情况下不执行任何操作。
* viewDidAppear：视图已完全过渡到屏幕上时调用。
* viewWillDisappear：视图被驳回后调用，覆盖或以其他方式隐藏。
* viewDidLoad：在视图加载后被调用，如果是通过代码创建的视图加载器，将会在loadView方法后被调用，如果是Nib视图输出，会在视图设置好之后被调用。
* alloc --> init --> loadView --> viewDidLoad --> viewWillAppear -->     viewDidAppear --> viewWillDisappear --> viewDidDisappear --> dealloc

<!--more-->
# APP 启动的过程
* 有 storyboard ：
    *  main 函数
    * UIApplicationMain
        * 创建 UIApplication 对象
        * 创建 UIApplication 的 delegate 对象
    * 根据 Info.plist 获得 Main.storyboard 的文件名，加载 Main.storyboard。
        * 创建 UIWindow
        * 创建和设置 UIWindow 的 rootViewController
        * 显示窗口
* 无 storyboard：
    * main 函数
    * UIApplicationMain
        * 创建 UIApplication 对象
        * 创建 UIApplication 的 delegate 对象
    * delegate 对象开始处理系统事件
        * 在 `application:didFinishLaunchingWithOptions:` 中创建 UIWindow
        * 创建和设置 UIWindow 的 rootViewController
        * 显示窗口
    
# 试着 scrollView 的contentsize
一般情况下可以在 viewDidLoad 中设置，但是在 autolayout 下，系统会在viewDidAppear 之前根据 subView 的constraint 重新计算 scrollView 的contenSize.

解决办法：
* 去除 autolayout 选项，自己手动设置 contentSize.
* 使用 autolayout，要么自己设置完subView的constraint，然后让系统根据constraint自动计算出contentSize。要么就在 viewDidAppear 里面自己手动设置 contenSize.

# UIView、UIWindow、CALayer
* UIView：属于 UIkit.framework 框架，负责渲染矩形区域的内容，为矩形区域添加动画，相应区域的触摸事件，布局和管理一个或多个子视图。

* UIWindow：属于 UIkit.framework 框架，是一种特殊的 UIView，通常在一个程序中只会有一个 UIWindow，但可以手动创建多个 UIWindow，同时加到程序中。UIWindow的主要作用：
    * 作为容器，包含app所要显然的所有视图。
    * 传递触摸消息到程序中的 view 和 其他对象。
    * 与 UIViewController 协同工作，方便完成设备方向旋转的支持。

* CALayer：属于 QuartzCore.framework，用来绘制内容，对内容进行动画处理依赖于UIView来进行显示，不能处理用户事件。
* UIView 和 CALayer 是相互依赖的，UIView 依赖 CALayer 提供的内容，CALayer 依赖 UIView 的容器显示绘制的内容。

# frame 和 bounds 有什么不同
* frame：该 view 在父 view 坐标系统中的位置和大小，参考点是父坐标系统。
* bounds：该 view 在本身坐标系中的位置和大小，参考点是本身坐标系统。

# UITableView 的性能优化
* 使用不透明视图。不透明视图可以提高渲染的速度。可以将cell及其子视图的 `opaque` 属性设置为 YES。
* 不要重复创建不必要的cell，使用重用cell。
* 不需要与用户交互时，使用 `CALayer`，将内容绘制到 `layer上`，然后对 cell 的 `contentView.layer` 调用 `addSublayer:` 方法。
* 提前计算并缓存行高
* 不要阻塞主线程。使用多线程，让子线程去请求网络。
* cell中的view尽量不要使用透明
* 减少子视图的层级关系

# layoutSubview 何时会被调用
* 初始化 init 方法不会触发。
* 滚动UIScrollView 会触发。
* 旋转屏幕触发。
* 改变 view 的触发，前提是 frame 改变了。
* 改变 UIView 的大小触发。

# 应用的生命周期
* `- (BOOL)application:(UIApplication )application willFinishLaunchingWithOptions: (NSDictionary )launchOptions`：告诉代理进程启动但还没有进入状态保存。
* `- (BOOL)application:(UIApplication )application didFinishLaunchingWithOptions: (NSDictionary )launchOptions`：告诉代理，启动基本完成程序准备开始运行。
* `- (void)applicationWillResignActive:(UIApplication *)application`：当应用程序将要进入非活动状态执行，此期间应用程序不接收消息或事件，比如来电话了。
* `- (void)applicationDidBecomeActive:(UIApplication *)application`：当应用程序进入活动状态执行。
* `- (void)applicationDidEnterBackground:(UIApplication *)application`：当程序被推送到后台的时候调用。所以要设置后台继续运行，则在这个函数里设置。
* `- (void)applicationWillEnterForeground:(UIApplication *)application`：当程序从后台将要进入前台时调用。
* `-(void)applicationWillTerminate:(UIApplication *)application`：当程序将要退出时调用，通常做一些保存数据和退出前的清理工作。

# load 、initialize
* load：
    * 当类对象被引入项目时，runtime 会向每一个类对象发送 load  消息。
    * load 方法会在每个类甚至分类被引入时调用一次，调用的顺序：父类优先于子类。
    * load 方法不会被类自动继承
    
* initialize:
    * 也是在第一次使用这个类的时候会调用这个方法。




