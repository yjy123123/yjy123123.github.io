---
title: 逆向Write up合集
date: 2021-03-06 15:06:06
tags: 逆向
description: CTF逆向题练手合集，之后可能会更多的总结知识点，持续更新。
---
## XCTF攻防世界

### open_source

这道题直接给了源码，读就行了，一开始以为文件名是32位hash，但是计算的话太大，也无法解密，所以突破口还是hash本身的计算，从之前的判断可以推出，其实(second % 17)就是等于8。

算出来还要转为16进制。


### logmein

也是考的阅读c的能力，拖到ida里面F5反编译，简单读一下，突破口应该在判断语句里面。

![](reverse-1.png)

处理之后提示数太大了，乍看一眼*(_BYTE *)不太理解什么意思，但是感觉long整数和字符串长度差不多，可以合理推测，可能需要对long型的数据一个字节一个字节的处理。

结果是我看错了，括号其实对应错了，(*((_BYTE *)&v7 + i % v6)，(_BYTE *)&v7是一个整体，实现的功能也是从long中依次取一个字节的数。

首先要知道long在机器中是如何存储的，才知道怎么取出，在汇编中，一般是小端存储，所以按地址取数是从低位开始往高位取，一个字节可以存储两个16进制数，所以一次取两个数，这也解释i % 7 的原因。

最开始想用模运算取数，还停留在10进制中，但是python实现16进制模运算有一个坑就是，会把%识别成格式化的符号，所以这了采用```x>>(8^i)&0xFF```来取数据。

最后16进制结果要转为ascii码。

这道题花了一点时间，我用python处理进制的转换非常的不熟悉，真的头大。

[这篇文章](https://reverseengineering.stackexchange.com/questions/13703/what-is-byte-usercall-eax-and-edi)中对*(_BYTE *)的解释：

```
This actually has two distinctive parts, evaluated one after the other.

First, (_BYTE *) casts a value or a register to be a byte pointer. This is similar to assigning the value to a C variable which is defined as byte *.

Second, * dereferences the address and retrieves the value in that address, value is assumed to be of the type of the pointer, in our case _BYTE.
```

拆解出来就是先将值转换为字节指针，之后再找地址对应值。

---
我觉得自己的解法太麻烦了，最后看了一下其他人的，有的v7反汇编出来就是一个字符串而不是long型，但是这个估计一开始会想不到小端存储，把v7换成字符串处理确实方便很多。

### insanity

先用ida看了一下，反编译出来的结果很简单，看一看有没有什么有效的字符信息。直接看到一个像flag的东西，我开始只提交了字符，没加前面数字，还好再尝试提交了一次，不然又掉坑里面了。

### getit

ida反编译main函数，感觉核心内容是对"/tmp/flag.txt"文件的写入，fputc做的就是写入操作，另外关注s和t两个参数，从长度来看，感觉又是和logmein一样的套路，但是简单很多，也没什么大小端。

但是总是提交不对，我还以为第二步文件写入打乱顺序了写入，可能要动态调试因为太麻烦了准备看一下到底怎么回事，结果是ida里面字符串只显示了```harifCTF{}```，掉了一个S，ghidra里面可以正常显示出，我不知道为什么，我差点骂出世界上最脏的脏话。不过这道题确实可以动态分析跑出来，更快一点。

### python-trade

这一道题是python反编译，```pip install uncompyle```，用uncompyle反编译出来直接是python代码了。

这里有一个点就是，base64解码出来的结果是bytes。

### re1

反编译之后看到strcmp，v9和v5，v5是用户输入，突破口就是v9了，顺着找过去是一串16进制数，分成了两段，直接16进制转ascii码看一下，是一串像flag的东西，但是顺序是反的，从程序逻辑来看就是flag，不知道考点是啥。

而且这道题在macos下会乱码。

### game

这道题感觉程序比较大，简单用ida看了一下，感觉没什么有效信息，所以还是运行玩一玩游戏，感觉这道题可能用动态分析工具比较好，现在是考虑修改内存把灯全部点亮。

### 小总结

做了好几道题，简单题基本上还是读C，要熟悉16进制的各种操作，而且本质就是通过加密算法写对应的解密算法。



