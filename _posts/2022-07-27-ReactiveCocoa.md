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

```c
- (void)createSignal{
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@8];
        [subscriber sendCompleted];
        return nil;
    }];
    [signal subscribeNext:^(id x) {
        NSLog(@"订阅者 %@",x);
    }];
}
```
### Subscription

订阅者是正在等待或能够等待信号事件的任何东西。在RAC中，订阅者表示为符合RACSubscriber协议的任何对象。
订阅是通过调用-subscribeNext:error:completed:或相应的方便方法创建的。从技术上讲，大多数RACStream和RACSignal操作符也创建订阅，
但这些中间订阅通常是实现细节。订阅保留它们的信号，并在信号完成或出错时自动清楚，订阅也可以手动清除。
```c
RACSignal *letters = [@"A B C D E F G H I" componentsSeparatedByString:@" "].rac_sequence.signal;

// Outputs: A B C D E F G H I
[letters subscribeNext:^(NSString *x) {
    NSLog(@"%@", x);
}];
```
### RACCommand

由RACCommand 类表示的命令创建并订阅响应某个操作的信号。这使得用户在与应用程序交互时很容易执行side-effecting 工作。通常触发命令的动作是由ui驱动的，
就像点击按钮一样。命令也可以根据信号自动禁用，这种禁用状态可以在UI中通过禁用与该命令相关的任何控件来表示。
```c
- (void)raccommand{
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        NSLog(@"执行命令");
        // 2.创建信号,用来传递数据
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            [subscriber sendNext:@"请求数据"];
            // 注意：数据传递完，最好调用sendCompleted，这时命令才执行完毕。
            [subscriber sendCompleted];
            return nil;
        }];
    }];
 
    // 订阅RACCommand中的信号
    [command.executionSignals subscribeNext:^(id x) {
        NSLog(@"command.executionSignals %@",x);
        [x subscribeNext:^(NSString *x) {
            NSLog(@"x subscribeNext %@",x);
        }];
    }];

    // 监听命令是否执行完毕,默认会来一次，可以直接跳过，skip表示跳过第一次信号。
    [[command.executing skip:1] subscribeNext:^(id x) {
        if ([x boolValue] == YES) {
            NSLog(@"正在执行");
        }else{
            NSLog(@"执行完成");
        }
    }];
    [command execute:@1];
}
```
### Connections

由RACMulticastConnection类表示的连接是在任意数量的订阅者之间共享的订阅。信号在默认情况下是冷的，
这意味着每次添加新订阅时它们就开始工作。这种行为通常是可取的，因为它意味着将为每个订阅者重新计算数据，
但如果信号有副作用或工作成本很高(例如，发送网络请求)，则可能会出现问题。

连接是通过RACSignal上的-publish或-multicast:方法创建的，并确保只创建一个底层订阅，
不管订阅了多少次连接。连接之后，连接的信号被称为热的，基础订阅将保持活动状态，直到对该连接的所有订阅被销毁。

```c
- (void)multicastConnection{
    __block int count = 0;
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        ++ count;
        NSNumber* number = [NSNumber numberWithInt:count];
        [subscriber sendNext:number];
        return nil;
    }];
    RACMulticastConnection *connect = [signal publish];
    [connect.signal subscribeNext:^(id x) {
        NSLog(@"订阅者 %@",x);
    }];
    [connect connect];
}
```
### Sequences

由RACSequence 类表示的序列是一个 pull-driven（拉驱动流），序列是一种集合，
在目的上类似于NSArray。与数组不同的是，序列中的值在默认情况下是惰性计算的(也就是说，只在需要它们的时候)，如果只使用序列的一部分，那么可能会提高性能。就像Cocoa 集合一样，序列不能包含 nil。RAC 为大多数Cocoa 的集合类添加了一个-rac_sequence 方法，允许它们被用作 racsequence。

```c
- (void)sequence {
    RACSequence *sequence = @[@1, @2, @3].rac_sequence;
    RACSignal *signal = sequence.signal;
    [signal subscribeNext:^(id  _Nullable x) {
        NSLog(@"%@", x);
    }];
}
```

## 冷热 Singal

### cold signal

大多数信号开始时都是“冷的”，这意味着它们在订阅之前不会做任何工作。在订阅时，信号或其订阅者可以执行side work，如登录到控制台、发出网络请求、更新用户界面等。还可以将side work 注入到信号中，它们不会立即执行，但会在以后的每次订阅中生效。

```c
_block unsigned subscriptions = 0;

RACSignal *loggingSignal = [RACSignal createSignal:^ RACDisposable * (id<RACSubscriber> subscriber) {
    subscriptions++;
    [subscriber sendCompleted];
    return nil;
}];

// Outputs:
// subscription 1
[loggingSignal subscribeCompleted:^{
    NSLog(@"subscription %u", subscriptions);
}];

// Outputs:
// subscription 2
[loggingSignal subscribeCompleted:^{
    NSLog(@"subscription %u", subscriptions);
}];
```

### hot signal

connection 是通过RACSignal 上的-publish 或-multicast: 方法创建的，并确保只创建一个底层订阅，不管订阅了多少次connection。连接之后，连接的信号被称为hot signal，基础订阅将保持活动状态，直到对该连接的所有订阅被销毁。

hot signal 可以有多个订阅者，是一对多，集合可以与订阅者共享信息；而cold signal 只能一对一，当有不同的订阅者，消息是重新完整发送。

```c
- (void)multicastConnection{
    // 假设在一个信号中计数，每次订阅一次都会触发计数，这样就会导致多次计数。
    // RACMulticastConnection: 解决重复处理的问题
    __block int count = 0;
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        ++ count;
        NSNumber* number = [NSNumber numberWithInt:count];
        [subscriber sendNext:number];
        return nil;
    }];
 
    RACMulticastConnection *connect = [signal publish];
    [connect.signal subscribeNext:^(id x) {
        NSLog(@"订阅者一信号 %@",x);
    }];
    [connect.signal subscribeNext:^(id x) {
        NSLog(@"订阅者二信号 %@",x);
    }];
    // 连接,激活信号
    [connect connect];
}
```