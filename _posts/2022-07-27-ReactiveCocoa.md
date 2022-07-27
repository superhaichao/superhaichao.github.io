---
title: 聊聊 ReactiveCocoa 响应式编程
author: superchao
date: 2022-07-27
categories: [Blogging, RAC]
tags: [RAC]
math: true
mermaid: true
---

## ReactiveCocoa

作为一名iOS开发者，你所编写的每一行代码都是针对某些事件的反应; 点击按钮、接收到的网络消息、属性改变(通过Key Value Observing)或通过CoreLocation改变用户位置都是很好的例子。 然而，这些事件都以不同的方式编码; 作为动作，委托，KVO，回调和其他。 ReactiveCocoa为事件定义了一个标准接口，因此可以使用一组基本的工具更容易地链接、过滤和组合事件。  

ReactiveCocoa结合了两种编程风格:  
* 函数式编程，利用高阶函数，即以其他函数为参数的函数;
* 响应式编程，关注数据流和变更传播;

由于这个原因，您可能听说过 ReactiveCocoa 被描述为函数响应式编程(或FRP)框架。  

### 与传统编程对比
