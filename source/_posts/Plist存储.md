---
title: 数据持久化保存——Plist存储
tags:
  - 数据持久化保存
category:
  - iOS开发
date: 2016-07-07 14:30:32
---



* Plist只能存储系统自带的一些常规的类, 也就是有writeToFile方法的对象才可以使用plist保存数据，例如：NSString、NSDictionary、NSArray、NSData、 NSNumber等类型

* 在存储之前要先找到沙盒路径

```objectivec
// NSUserDomainMask 在用户目录下查找
// YES 代表用户目录的~
// NSDocumentDirectory 查找Documents文件夹
// 建议使用如下方法动态获取
NSString *doc = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];

// 拼接文件路径
NSString *path = [doc stringByAppendingPathComponent:@"abc.plist"];
NSLog(@"%@", path);
```

* 写入文件

```objectivec
NSArray *arr = @[@"lnj", @"28"];
[arr writeToFile:path atomically:YES];

NSDictionary *dict = @{@"name": @"lnj", @"age":@"28"};

// 调用writeToFile将数据写入文件
[dict writeToFile:path atomically:YES];
```

> 注意：自定义的对象 不能直接保存到plist文件中

* 读取文件

```objectivec
NSString *doc = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];

NSString *path = [doc stringByAppendingPathComponent:@"abc.plist"];

// 读取数据
NSDictionary *dict = [NSDictionary dictionaryWithContentsOfFile:path];
NSLog(@"%@", dict);
```
