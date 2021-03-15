---
layout: post
title: "AFNetworkReachability"
# background-image: ""
date: 2021-2-23
published: true
category: iOS
tags:
        - AFNetworking
---

基础用法
```objc
    AFNetworkReachabilityManager *manager = [AFNetworkReachabilityManager manager];
    [manager setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
        switch (status) {
            case AFNetworkReachabilityStatusUnknown:
                NSLog(@"不明网络");
                break;
            case AFNetworkReachabilityStatusNotReachable:
                NSLog(@"没有网络");
                break;
            case AFNetworkReachabilityStatusReachableViaWWAN:
                NSLog(@"蜂窝网络");
                break;
            case AFNetworkReachabilityStatusReachableViaWiFi:
                NSLog(@"WiFi"); //光纤
                break;
            default:
                break;
        }
    }];
    [manager startMonitoring];
```

底层为原生SystemConfigration库内的`SCNetworkReachabilityRef`


`startMonitoring`内做了什么？
```objc
- (void)startMonitoring {
    [self stopMonitoring];
    if (!self.networkReachability) {
        return;
    }
    __weak __typeof(self)weakSelf = self;
    // 创建一个在每次网络状态改变时的回调
    AFNetworkReachabilityStatusBlock callback = ^(AFNetworkReachabilityStatus status) {
        __strong __typeof(weakSelf)strongSelf = weakSelf;
        /*
         每次回调被调用时
         重新设置 networkReachabilityStatus 属性
         再调用 networkReachabilityStatusBlock
         */
        strongSelf.networkReachabilityStatus = status;
        if (strongSelf.networkReachabilityStatusBlock) {
            strongSelf.networkReachabilityStatusBlock(status);
        }
    };
 
    SCNetworkReachabilityContext context = {0, (__bridge void *)callback, AFNetworkReachabilityRetainCallback, AFNetworkReachabilityReleaseCallback, NULL};

    // 为SCNetworkReachability添加回调函数 在状态变化的时候会调用这个回调
    // 这个方法中还要一个回调函数，和一个 SCNetworkReachabilityContext 。回调函数用来当网络状态发生变化时，让系统进行回调。
    SCNetworkReachabilitySetCallback(self.networkReachability, AFNetworkReachabilityCallback, &context);

    // 在设置完了回调函数后，将networkReachability加入到runloop中，达到实时监听的目的
    // 这里是kCFRunLoopCommonModes,这样当用户操作UI的时候，也会继续监听。和NSTimer的使用类似，将定时器添加到这个模式的runloop中时,滑动事件就不会中断定时器的计时。
    SCNetworkReachabilityScheduleWithRunLoop(self.networkReachability, CFRunLoopGetMain(), kCFRunLoopCommonModes);

    // 在异步线程 发送一次当前的网络状态。
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0),^{
        SCNetworkReachabilityFlags flags; // status
        if (SCNetworkReachabilityGetFlags(self.networkReachability, &flags)) {
            AFPostReachabilityStatusChange(flags, callback);
        }
    });
}
```

监听网络权限状态
```objc
- (void)cellularData {
    CTCellularData *cellularData = [[CTCellularData alloc] init];
    cellularData.cellularDataRestrictionDidUpdateNotifier = ^(CTCellularDataRestrictedState state){
        switch (state) {
            case kCTCellularDataRestricted:
            {
                NSLog(@"网络权限被拒绝");
            }
                break;
            case kCTCellularDataNotRestricted:
            {
                NSLog(@"网络权限正常，可以监控网络状态");
            }
            case kCTCellularDataRestrictedStateUnknown:
            {
                NSLog(@"未知状态");
            }
                
            default:
                break;
        }
    };
}
```