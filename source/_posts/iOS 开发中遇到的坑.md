---
title: iOS 开发中遇到的坑
tags:
  - 笔记
category:
  - iOS开发
date: 2016-09-20 14:39:33
---

# 记录 iOS 开发中遇到的坑，持续更新......
## iOS 10 相关

### 权限描述 crash 
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

## Xcode

### 无权限访问此文件
**问题描述：**
   当修改项目中的 plist 文件后运行项目，会出现以下错误：

![](https://o9xwn216o.qnssl.com/blog-img/1473670178405.png)

**原因分析**
猜想可能是缓存内的文件和当前编译文件冲突

**解决办法**
1. cmd + shift + k 清空 -—> cmd + r
2. 方法一有时候会不管用，如果不管用，就 Project-->Build Setting中 修改这一项，变成Default Compiler（Apple LLVM 6.0） 

![](https://o9xwn216o.qnssl.com/blog-img/1473670782316.png)

## AutoLayout
在使用 AutoLayout布局时，先制定一套约束如下：



### 横屏状态栏不显示
横屏后状态栏不显示了,这是因为iOS系统(好像是iOS8之后)在视图横屏的时候默认把状态栏隐藏掉了，解决方案：

1. 在plist文件中将 View controller-based status bar appearance 设置为NO

2. 在application:didFinishLaunchingWithOptions:中添加下面代码

```objectivec
[[UIApplication sharedApplication] setStatusBarHidden:YES withAnimation:UIStatusBarAnimationNone];

[[UIApplication sharedApplication] setStatusBarHidden:NO withAnimation:UIStatusBarAnimationNone];
```

## 横屏竖屏判断
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

UIInterfaceOrientationPortraitUpsideDown = UIDeviceOrientationPortraitUpsideDown,
    
UIInterfaceOrientationLandscapeLeft      = UIDeviceOrientationLandscapeRight,

UIInterfaceOrientationLandscapeRight     = UIDeviceOrientationLandscapeLeft
```

https://o9xwn216o.qnssl.com/blog-img/1476453944034.png


