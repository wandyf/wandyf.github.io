---
layout: post
title: MVVM-C With Swift
# background-image: 
date: 2020-04-22
published: true
category: 架构
tags:
        - swift
---

## Model-View-ViewModel-Coordinator

![MVVM-C结构图](https://raw.githubusercontent.com/wandyf/wandyf.github.io/master/assets/img/mvvm-c.png "MVVM-C 结构图")


- View Controller不会引用其他的View Controller和ViewModel
- View Controller通过delegate机制，将View Action告知Coordinator
- Coordinator根据View Action来调度其他的View Controller并填充Model
- View Controller的层级关系是由Coordinator进行管理的，而不是由View Controller来决定
- Coordinator持有Model并知晓View Controller的层级结构，因此能够为ViewModel提供所需的Model对象
- Coordinator以增加Controller接口为代价，降低View Conttoller之间的耦合


---
# 构建

- Model由View Controller创建，但由ViewModel持有
- View由View Controller创建，但不为其提供数据，只掌控View的层级关系
- ViewModel在创建ViewController时一并创建
- ViewModel将View Controller创建的View绑定到ViewModel相应的属性上

# Model的更新

- View Controller接收View Action，但自身不做处理
- View Controller将View Action转达给ViewModel(通过ViewModel暴露的方法)
- ViewModel更改自身内部的状态和Model(如果Model需要更新的话)

# View的更新

- View Controller不监听Model
- ViewModel负责观察Model并将Model的变化通知到View Controller
- View Controller订阅View Model的变更(通过响应式框架或其他方式)
- View Controller通过订阅View Model去变更View的层级关系
- ViewModel将Model的View Action发送给Model,并在Model发生实际变化后通知给相关订阅者

# View State

- View Controller不存储任何的View State
- View State存储于View本身或ViewModel中
- ViewModel中的View State变化会被其订阅者View Controller观察到
- View Controller无法区分通知是由于Model的变更还是View State的变更
- View Controller的层级关系有Coordinator控制

# Test

- ViewModel与View和View Controller解耦
- ViewModel可以进行单独的接口测试
- View Controller中的初始化创建和与Coordinator交互的部分需要单独进行测试

