title: 适配器模式
date: 2015-06-17 08:27:32
categories: 设计模式
tags: [设计模式,适配器]
---

## 理解
将一个类的接口变换成客户端所期待的另一个接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作。


**为什么需要它**

1. 你想使用一个已经存在的类，而它的接口不符合你的需求
2. 你想创建一个可以复用的类，该类可以与其他不相关的类或不可预见的类（即那些接口可能不一定兼容的类）协同工作。
3. （仅适用于对象Adapter）你想使用一些已经存在的子类，但是不可能对每一个都进行子类化以匹配它们的接口。对象适配器可以适配它的父类接口。