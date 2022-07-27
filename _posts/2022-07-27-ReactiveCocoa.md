---
title: 聊聊 ReactiveCocoa 响应式编程
author: superchao
date: 2022-07-27 11:33:00 +0800
categories: [Blogging, RAC]
tags: [typography]
math: true
mermaid: true
image:
  path: /commons/devices-mockup.png
  width: 800
  height: 500
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

## 聊聊 ReactiveCocoa 响应式编程

作为一名iOS开发者，你所编写的每一行代码都是针对某些事件的反应; 点击按钮、接收到的网络消息、属性改变(通过Key Value Observing)或通过CoreLocation改变用户位置都是很好的例子。 然而，这些事件都以不同的方式编码; 作为动作，委托，KVO，回调和其他。 ReactiveCocoa为事件定义了一个标准接口，因此可以使用一组基本的工具更容易地链接、过滤和组合事件。  

