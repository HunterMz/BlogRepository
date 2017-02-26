---
title: iOS 开发中遇到的坑
tags:
  - 笔记
category:
  - iOS开发
date: 2016-09-20 14:39:33
---

# 记录 iOS 开发中遇到的坑，持续更新.....
## iOS10 权限描述 crash 
**问题描述：**当使用 iOS10 运行项目会 crash，报错信息如下：

<font color=red>**This app has crashed because it attempted to access privacy-sensitive data without a usage description.  The app's Info.plist must contain an NSContactsUsageDescription key with a string value explaining to the user how the app uses this data.**</font>

**原因分析：**

在 iOS10 中，如果有访问系统资源，例如：照相机、相册、麦克风、通讯录等，需要在 info.plist 文件中添加描述字段

**解决方案：**

<!--more-->

```
// 使用照相机
<key>NSCameraUsageDescription</key>    
	<string>此应用会访问您的相机</string>

// 使用相册
<key>NSPhotoLibraryUsageDescription</key>
    <string>此应用会访问您的相册</string>
// 使用通讯录
<key>NSContactsUsageDescription</key>    
	<string>此应用会访问您的通讯录</string>

// 使用麦克风
<key>NSMicrophoneUsageDescription</key>    
	<string>此应用会访问您的麦克风</string>
```

## 无权限访问此文件
**问题描述：**
   当修改项目中的 plist 文件后运行项目，会出现以下错误：

![](https://o9xwn216o.qnssl.com/blog-img/1473670178405.png)

**原因分析**
猜想可能是缓存内的文件和当前编译文件冲突

**解决办法**
1. cmd + shift + k 清空 -—> cmd + r
2. 方法一有时候会不管用，如果不管用，就 Project-->Build Setting中 修改这一项，变成Default Compiler（Apple LLVM 6.0） 

![](https://o9xwn216o.qnssl.com/blog-img/1473670782316.png)

## Xcode编译时候出现各种诡异的问题
请到这个`/Users/用户名/Library/Developer/Xcode/DerivedData `目录下清理下Xcode缓存,我的问题是，编译时候报文件找不到，但是可以跑起来，`cmd + shift + c `了好几次都不行，然后到缓存目录，全部清理掉，就可以了。。。


## 横屏状态栏不显示
横屏后状态栏不显示了,这是因为iOS系统(好像是iOS8之后)在视图横屏的时候默认把状态栏隐藏掉了，解决方案：

1. 在plist文件中将 View controller-based status bar appearance 设置为NO

2. 在application:didFinishLaunchingWithOptions:中添加下面代码

```objectivec
[[UIApplication sharedApplication] setStatusBarHidden:YES withAnimation:UIStatusBarAnimationNone];

[[UIApplication sharedApplication] setStatusBarHidden:NO withAnimation:UIStatusBarAnimationNone];
```

## 手机平放横屏竖屏判断
iOS 可以检测屏幕方向，如下枚举

```objectivec
UIDeviceOrientationUnknown            // 方向未知    UIDeviceOrientationPortrait           // Device oriented vertically, home button on the bottom

UIDeviceOrientationPortraitUpsideDown // Device oriented vertically, home button on the top

UIDeviceOrientationLandscapeLeft      // Device oriented horizontally, home button on the right

UIDeviceOrientationLandscapeRight     // Device oriented horizontally, home button on the left

UIDeviceOrientationFaceUp             // Device oriented flat, face up

UIDeviceOrientationFaceDown  
```
但是当手机平放时就无法判断是横屏还是竖屏了。准确的说是不知道从`横屏进入平放`还是`从竖屏进入平放`。假设你的横屏和竖屏 UI 变化比较大，那么你的设备	`从横屏变为平放` 和 `从竖屏变为平放`就尤为重要了。

终于找到一种方法，可以解决问题了....

通过判断 statusBarOrientaion 的方向。

```objectivec
UIInterfaceOrientationUnknown            = UIDeviceOrientationUnknown,
 
UIInterfaceOrientationPortrait           = UIDeviceOrientationPortrait,

UIInterfaceOrientationPortraitUpsideDown = **UIDeviceOrientationPortraitUpsideDown,
    
UIInterfaceOrientationLandscapeLeft      = UIDeviceOrientationLandscapeRight,

UIInterfaceOrientationLandscapeRight     = UIDeviceOrientationLandscapeLeft
```

## 判断当前控制器是 push 进来的还是 present 进来
```objectivec
if (self.navigationController.viewControllers.count > 1) {
	// push 
	[self.navigationController popViewControllerAnimated:YES];
}else {
	// present
	[self dismissViewControllerAnimated:[NSUserDefaults standardUserDefaults].userAnimated completion:nil];
}
```

## 审核
### Missing Push Notification Entitlement
> Missing Push Notification Entitlement - Your app includes an API for Apple's Push Notification service, but the aps-environment entitlement is missing from the app's signature. To resolve this, make sure your App ID is enabled for push notification in the Provisioning Portal. Then, sign your app with a distribution provisioning profile that `includes the aps-environment entitlement`. This will create the correct signature, and you can resubmit your app. See "Provisioning and Development" in the Local and Push Notification Programming Guide for more information. If your app does not use the Apple Push Notification service, no action is required. You may remove the API from future submissions to stop this warning. If you use a third-party framework, you may need to contact the developer for information on removing the API.

Xcode 7以后的坑，解决办法：

![](http://o9xc0bh9t.bkt.clouddn.com/14834937846063.jpg)

### 内购商品超过99.99美元被拒绝
iTunesConnect上允许你建立超过99.99美元的消耗型商品，但是审核的时候默认是不允许超过99.99美元的消耗型商品。

## IDFA
iOS开发中，我们经常需要获取设备的唯一表示符。以前可以通过获取UDID来作为设备的唯一标识。但是2012年5月1日后，苹果开始禁止应用获取设备的UDID。那只能通过获取IDFA来识别设备了。

**问题描述：**
	使用以下代码获取设备的IDFA
	
```
[[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString];
```
在运行阶段没有任何问题，但是当把应用传到TestFlight进行测试的时候发现，每次杀掉进程重新启动应用获取到的IDFA都是不一样的。当应用通过审核，上传到App Store时，又恢复正常。


	

