---
title: 聊聊 ReactiveCocoa 响应式编程
author: superchao
date: 2022-07-27 00:08:00 +0800
categories: [Blogging, Reactive]
tags: [Reactiveß]
img_path: /react/
---

作为一名iOS开发者，你所编写的每一行代码都是针对某些事件的反应，点击按钮、接收到的网络消息、属性改变(通过Key Value Observing)或通过 CoreLocation 改变用户位置都是很好的例子。 然而，这些事件都以不同的方式编码， 作为动作，委托，KVO，回调和其他。 ReactiveCocoa 为事件定义了一个标准接口，因此可以使用一组基本的工具更容易地链接、过滤和组合事件，相比之前的方式更加简单，ReactiveCocoa结合了两种编程风格:  
* 函数式编程，利用高阶函数，即以其他函数为参数的函数;
* 响应式编程，关注数据流和变更传播;

由于这个原因，您可能听说过 ReactiveCocoa 被描述为函数响应式编程(或FRP)框架。  

## 与传统编程对比

传统给按钮添加事件方式如下：
```c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view from its nib.
    
    [self.button addTarget:self action:@selector(buttonClick) forControlEvents:UIControlEventTouchUpInside];
}

- (void)buttonClick {
    
}

```
使用RAC 给按钮添加事件方式如下：
```c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view from its nib.
    self.button.rac_command = [[RACCommand alloc] initWithSignalBlock:^(id _) {
        NSLog(@"button was pressed!");
        return [RACSignal empty];
    }];
}
```
相比传统方式使用RAC 更加便捷，同理为了监控TextField 文本长度也不需要实现代理方法：
```c
[self.usernameTextField.rac_textSignal subscribeNext:^(id x) {
  NSLog(@"%@", x);
}];
```
## 切入正题 ReactiveObjC
RAC是一个将函数响应式编程范式带入iOS 的开源库，其兼具函数式与响应式的特性。它是由Josh Abernathy和Justin Spahr-Summers当初在
开发GitHub for Mac 过程中创造的，灵感来源于Functional Reactive Programming 。ReactiveCocoa-简称为RAC，现在可分为OC版本-ReactiveObjC和
swift版本-ReactiveSwift。
![Desktop View](racoverview.png)

### RACSignal

这是RAC 中最基本的一个概念，中文名叫做“信号”，由RACSignal 类表示的信号，RACSignal 是 push-driven 流，信号通常表示将来要交付的数据。
当工作被执行或数据被接收时，值被发送到信号上，将它们推送给任何订阅者。用户必须订阅信号才能访问其值。
信号向其订阅者发送三种不同类型的事件:
* next 下一个事件提供流中的新值。RACStream方法只对这种类型的事件进行操作。与Cocoa集合不同的是，信号包含nil是完全有效的。
* error 事件指示在信号完成之前发生了错误。该事件可能包含一个NSError对象，用于指示发生了什么错误。错误必须特殊处理——它们不包含在流的值中。
* completed 事件指示信号成功完成，并且不再向流添加任何值。完成必须特殊处理—它不包含在值流中。

信号的生命周期包括任意数量的下一个事件，然后是一个错误或完成事件(但不是两个都有)。

### Subscription

订阅者是正在等待或能够等待信号事件的任何东西。在RAC中，订阅者表示为符合RACSubscriber协议的任何对象。
订阅是通过调用-subscribeNext:error:completed:或相应的方便方法创建的。从技术上讲，大多数RACStream和RACSignal操作符也创建订阅，
但这些中间订阅通常是实现细节。订阅保留它们的信号，并在信号完成或出错时自动清楚，订阅也可以手动清除。
