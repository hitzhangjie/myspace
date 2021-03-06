---
layout: post
title: kref引用计数与kobject对象管理
description: "看完kref/kobject这几篇文档，更深地明白了一个道理，“能工模型，巧匠窃意”、“无招胜有招”，编程思想和编程工具是相辅相成的，前者帮助完善后者，后者便于更简单地推广前者。纵使是c语言这样的过程式编程语言，在牛人手里也可以提炼面向对象的精髓来建构更复杂的软件世界。"
date: 2022-07-07 22:20:07 +0800
tags: ["kref","kobject","refcount","destructor","smartpointer"]
categories: ["linux内核"]
toc: true
hide: false
---

## kref

kref可以为你自定义的结构体提供一个引用计数器，kobject也可以实
现该功能，但是kobject比较复杂，如果只是提供一个简单的引用计数器的话，应该使用
kref而不是kobject。

kref可以嵌入在我们定义的结构体struct中，当我们初始化一个结构体时通过kref_init对其进行初始化（引用计数为1），当我们引用这个struct时需要通过kref_get来增加其引用计数，而当我们不再引用这个struct时，我们可以通过kref_put来减少引用计数，同时还可以提供一个data_release的函数，当引用计数为0时该函数就会执行。

kref非常类似于c++中的智能指针的功能，gcc编译期对c语言也增加了一些类似的属性扩展，允许在变量作用域结束时执行注册的函数。可见，自定义类型中通过恰当地使用kref，我们就可以实现近似上述c++智能指针等高阶玩法。

see [kref.rst](https://sourcegraph.com/github.com/torvalds/linux/-/blob/Documentation/core-api/kref.rst)

## kobject

kobject又是什么呢，在面向对象领域中，对象有继承关系，派生对象需要实现抽象基类的方法，对象在没有被引用时也应该被自动销毁（联想c++析构函数）等。面向对象的那些思想在内核里面又是怎么样一种表现形式。

无招胜有招，c虽然是过程式编程语言，但是其依然可以写出面向对象的代码来对完成对大型软件项目的设计构建。

我们一般将kobject嵌入自定义的类型struct中来使用，同时还有对应的一个ktype：

- kobject，具备了引用计数功能，通过kobject_init/get/put操作可以对引用计数进行操作，另外kobject还有parent指针用来构建对象间的层级关系；
- ktype，用来描述每个kobject对象引用计数减为0时应该对这个包含kobject成员的struct类型执行何种操作，比如如何清理、释放之类的；

ps：kset可以看做是一个集合，用来管理一系列的kobject，使用场景见kobject.rst。

see [kobject.rst](https://sourcegraph.com/github.com/torvalds/linux/-/blob/Documentation/core-api/kobject.rst)

## 总结

看完这几篇文档，更深地明白了一个道理，“能工模型，巧匠窃意”、“无招胜有招”，编程思想和编程工具是相辅相成的，前者帮助完善后者，后者便于更简单地推广前者。纵使是c语言这样的过程式编程语言，在牛人手里也可以提炼面向对象的精髓来建构更复杂的软件世界。