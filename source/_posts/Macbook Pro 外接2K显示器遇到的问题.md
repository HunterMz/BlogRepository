---
title: Macbook Pro 外接2K显示器遇到的问题及解决办法
tags:
  - 经验
category:
  - 效率
date: 2017-09-22 20:00:33
---

最近跟公司申请了一台2K显示器([DELL 2525H](https://item.jd.com/1453819.html))配合我的MacBook Pro 2014年低配 (系统：10.13 beta)使用。想愉快的开启双屏模式，没想到连接后出现不少问题。把外接显示屏分辨率设置为默认 `2560 * 1440` 后字体变小，看着很难受，选择其他分辨率气体变大了但是又开始变的模糊，同样看着不舒服。于是开始在网上查找并尝试各种解决办法。最终还是被我找到了。

# 问题原因：Macbook 外接显示器默认为 TV 模式，TV 渲染模式下，文字效果非常差。

# 解决办法 

* 下载脚本

	* 首先现在一个脚本: [patch-edid.rb](https://gist.github.com/adaugherity/7435890)，默认会下载到`Downloads` 目录下。

	* 打开终端，执行下面两行代码：

		```vim
		cd /Users/username/Downloads
		ruby patch-edid.rb
		```
		> 注：运行patch-edid.rb脚本后，会产生一个DisplayVendorID-1xxx文件，xxx是编号。
* 关闭 rootless 功能：

 	* 关机 ——> 按住 `command + R` 开机，待屏幕出现苹果Logo 和 进度条的时候松开。
 	* 打开终端，执行以下指令：
 	
 		```vim
 		csrutil disable
 		```
 	* 重启
* 拷贝刚才得到的文件夹 `DisplayVendorID-1xxx` 至 `/System/Library/Displays/Contents/Resources/Overrides`

* 下载安装 SwitchResX 
	* 官网下载即可，可使用7天[SwitchResX](http://www.madrau.com)
	* 设置
	![](http://o9xc0bh9t.bkt.clouddn.com/15060653516657.jpg)
	输入如下参数：
	![](http://o9xc0bh9t.bkt.clouddn.com/15060653799030.jpg)
	之后勾选这个参数，开启你的双屏之旅吧。
	![](http://o9xc0bh9t.bkt.clouddn.com/15060655765360.jpg)






