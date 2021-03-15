---
layout: post
title: "MethodSwizzle"
# background-image: ""
date: 2021-3-2
published: true
category: iOS
tags:
        - runtime
---

### 定义  

*利用OC的`Runtime`特性，动态改变`SEL(方法编号)`和`IMP(方法实现)`的对应关系，达到OC方法调用流程改变的目的。主要用于OC方法。*  

![2021030401.jpg](https://raw.githubusercontent.com/wandyf/wandyf.github.io/master/assets/img/2021030401.jpg "2021030401.jpg")  


### 代码演示：  

*替换系统的[NSURL URLWithString:]方法，实现自动对中文进行URL编码*  
```objc
#import "NSURL+MSURL.h"
#import <objc/runtime.h>

@implementation NSURL (MSURL)

+ (void)load {
    // 获取原方法
    Method URLWithString = class_getClassMethod(self, @selector(URLWithString:));
    // 获取自定义方法
    Method MS_URLWithString = class_getClassMethod(self, @selector(ms_URLWithString:));
    // 交换方法实现
    method_exchangeImplementations(URLWithString, MS_URLWithString);
}

+ (instancetype)ms_URLWithString:(NSString *)string {
    NSURL *url = [NSURL ms_URLWithString:string];
    if (!url) {
        string = [string stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
    }
    return [NSURL ms_URLWithString:string];
}

@end
```