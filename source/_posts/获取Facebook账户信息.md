---
title: 获取Facebook账号信息
tags:
  - 笔记
category:
  - iOS开发
date: 2016-12-13 18:00:33
---

## 背景
最近公司有一个`绑定Facebook账号`的需求。最先想到的肯定是集成`FacebookSDK`，这个实现起来很简单，因为Facebook的文档写的实在太详细了。但是我做的是个SDK，导入太多第三方库不太好，而且需要在项目里配置一堆东西。把需求细化，其实只需要获取用户的`Facebook ID`。在老大的提示才想到苹果早就把Facebook集成到系统中了，只要在【设置】->【Facebook】中登录了账号，就可以在用户授权的情况下获取用户的Facebook ID了。

## 操作
1.  [Facebook 开发者中心](https://developers.facebook.com)注册应用，注册成功后你会获得一个 `AppIdKey`。

2.  新建一个工程，`bundle ID`要和Facebook上注册的保持一致。

3. 在需要使用的位置导入头文件

	```ObjectiveC
#import <Accounts/Accounts.h>
#import <Social/Social.h>
	```

4. 拿到Facebook的 `oauthToken`

	```ObjectiveC
+ (void)getFacebookTokenWithAppIdKey:(NSString *)appIdKey completion:(GetFaceBookBlock)completion {
    
    ACAccountStore *accountStore = [[ACAccountStore alloc]init];
    ACAccountType *FBaccountType= [accountStore accountTypeWithAccountTypeIdentifier:ACAccountTypeIdentifierFacebook];
    NSDictionary *params = @{ACFacebookAppIdKey:appIdKey, ACAccountTypeIdentifierFacebook:@[]};
    [accountStore requestAccessToAccountsWithType:FBaccountType options:params completion:^(BOOL granted, NSError *error) {
       	if (granted) {
            ACAccount *account = accountStore.accounts.lastObject;
            completion(account, nil);
        }else {
            completion(nil,error);
        }
    }];
}	
	```
	
5. 获取Facebook 的 `userID`
	这一步猜到一个巨坑，因为在上一步会得到一个 `account`，然后你会发现这个对象不仅包含了你的`username` 还包含了一个 `identifier`
	
	![](http://o9xc0bh9t.bkt.clouddn.com/14816220995122.jpg)

	想当然的以为这个就是FacebookID，但是隐约记得以前用FacebookSDK做测试时得到的ID不是长这样子的，然后去验证了下，果然不是......接着就查文档啊，苹果和Facebook开发文档都查过了，任然一无所获。万般无奈，去stack overflow 上请教大神，很快查到了，再一次证明了stackoverflow对程序员的重要性。原来还要用前面得到的`oauthToken`去Facebook做验证，验证的接口是`https://graph.facebook.com/me`

	```ObjectiveC
[AGBindFacebookAccount getFacebookTokenWithAppIdKey:@"123456" completion:^(ACAccount *account, NSError *error) {
        
        NSURL *requestURL = [NSURL URLWithString:@"https://graph.facebook.com/me/friends"];
        SLRequest *request = [SLRequest requestForServiceType:SLServiceTypeFacebook requestMethod:SLRequestMethodGET URL:requestURL parameters:nil];
        request.account = account;
        
        [request performRequestWithHandler:^(NSData *data, NSHTTPURLResponse *response, NSError *error) {
            if(error) {
                NSLog(@"error from get%@",error);
            } else {
                // 你要的 FacebookID 在 data 里
                NSLog(@"%@", data);
            }
        }];
    }];
	```


