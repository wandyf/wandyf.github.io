---
layout: post
title: "Http,Https与AFSecurityPolicy"
# background-image: ""
date: 2021-2-20
published: true
category: iOS
tags:
        - AFNetworking
---

Http版本
- 0.9 实现Get请求
- 1.0 增加POST HEAD等
  -   缺点:每一个TCP只能发送一次请求，每次连接耗费性能
- 1.1 增加keep-alive  
  - 短时间内保持连接，超时再关闭连接   
  - 客户端可以同时发送多个请求，但服务器只能依次响应
- 2.0  
  - header压缩 gzip
  - 多工 多个请求同时响应 防止对头堵塞
  - 自推送 建立连接后服务器主动推送数据
  - 无状态

Http缺点
- 内容明文传输，不安全，易被监听，不安全，不稳定(丢数据无法找回)
- 不验证身份,任何人都可以访问,DoS攻击
- 数据完整性无法验证，可以篡改内容

Https
- http+加密+认证+完整保护
- 加密 (对称+非对称)
  - 对称加密(内容)
  - 非对称加密(RSA，加密对称加密的秘钥)  
  - CA证书，收费，无需额外操作    自签证书，免费，需要配置


AFSecurityPolicy  
- 作用是验证`HTTPS`证书是否有效
- 使用方法：
```objc
  NSString *cerPath = [[NSBundle mainBundle] pathForResource:@"https" ofType:@"cer"];
  NSData *data = [NSData dataWithContentsOfFile:cerPath];
  NSSet *cerSet = [NSSet setWithObject:data];  
  AFSecurityPolicy *security = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModeCertificate withPinnedCertificates:cerSet];
  [AFSecurityPolicy defaultPolicy];
  security.allowInvalidCertificates = YES;
  security.validatesDomainName = NO;
```

AFN证书验证流程
- 收集公钥集合
- 收到身份验证请求
  - `-(void)URLSession:didReceiveChallenge:completionHandler`
  - 根据配置判断是否验证服务器证书
  - 根据配置判断是否验证域名
  - 评估指定证书和策略的信任度：`AFServerTrustlsValid`
  - 证书评估通过，继续判断`Pinning Mode`:
    - `AFSSLPinningModeNone` 无条件信任，不允许，返回NO
    - `AFSSLPinningModeCertificate` 对比服务器证书和本地证书
    - `AFSSLPinningModePublickey` 对比服务证书和本地证书公钥
    - 是否包含公钥
