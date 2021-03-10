---
title: Unicorn简单上手记录
date: 2021-02-27 17:55:05
tags: 底软
description: 未完待续...
---

Unicorn是一个CPU模拟器，主要作用就是跨平台模拟执行，Unicorn是基于QEMU实现的，但是Unicorn仅关注CPU的模拟，并不关注其他外设，官方文档中说明Unicorn允许没有上下文的情况下模拟原始代码。其他Unicorn与QEMU的对比详情参考[官方文档](https://www.unicorn-engine.org/docs/beyond_qemu.html)。


在Unicorn之上继续发展就是[qiling](https://qiling.io)框架，操作系统级的模拟器，之后应该会学习，到时候再更一篇相关博客。

目前基本上Unicorn的引导教程都是基于[Unicorn Engine tutorial](http://eternal.red/2018/unicorn-engine-tutorial/)这篇文章。

###下载

[编译安装](https://www.unicorn-engine.org/docs/)

基本上类似的开源软件官方文档有非常详细的安装编译教程，选一种适合自己环境的方法就可以了。如果使用Python的模块的话，直接通过pip3下载，不过貌似安装麒麟框架的时候已经下好了，pip应该是最便捷的方法了。

###Task1

1.用的Pycharm，一开始并没有办法识别unicorn的库，我还以为安装失败了，其实因为默认的依赖库里面没有，更换成本地的依赖库就行了，不知道为什么明明重新设置了默认依赖，每次新项目都要手动更改一下

2.出现编码错误,因为我直接参考的原教程里面读取文件的方法```with open(name) as f:```,这里最好修改为以二进制格式读取文件```with open(name,"'rb') as f:```

```
UnicodeDecodeError: 'utf-8' codec can't decode byte 0x90 in position 24: invalid start byte
```

3.第二部分提升速度需要分析递归函数，入口就是0x400670，这一块分析有点难，文章提供的solve3.py里面有每一个步骤的备注，非常详细，但是分析二进制文件本身对我来说有点难，还没有理清楚，代填坑。

---

基本步骤：


1.分配内存空间

2.开始模拟，找起始地址和终止地址

3.通过输出定位错误，是因为BSS段没有分配内存引起的错误，跳过所有出错的指令


---
未完待续




###小结

Unicorn需要手动的分配内存，但是比较灵活，可以从任意地址开始执行代码，不依赖上下文。