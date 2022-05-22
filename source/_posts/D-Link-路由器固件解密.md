---
title: D-Link 路由器固件解密
date: 2021-05-28 10:31:01
tags: 路由器
description: 跟着一篇博客尝试解密了一下D-Link的DIR-878的路由器固件。
---



### 写在前面

1. D-Link有一个专门提供固件下载的网站，里面基本上什么类型的固件都有，然后比较早版本的固件都是没有加密的，但是之后的固件基本上都进行了加密，用binwalk没办法解包。

固件下载🔗：https://tsd.dlink.com.tw

2. 这次参考的博客是[MINDSHARE: DEALING WITH ENCRYPTED ROUTER FIRMWARE](https://www.zerodayinitiative.com/blog/2020/2/6/mindshare-dealing-with-encrypted-router-firmware)

   可以解密的固件包括DIR-882、DIR-878和DIR-867的固件。基本原理就是找到一个包含解密功能的未加密固件，用这个解密的模块去解密之后的固件。

   感觉可能这个方法通用性没那么好，但是还是有点想动手体验一下，因为操作起来还是比较简单的。
   
3. 面试的时候（IoT方向）真的很多公司会问对加密固件怎么分析。



### 解密固件

#### 下载固件

这里需要的固件是DIR-882的v1.04B02版本的固件，因为这个固件是一个加密和未加密固件之间过渡版本，里面包含解密模块，但是固件本身并没有加密。

开始直接在下载固件的网站里面找，结果没有找到，后来发现这个v1.04B02，包含在v1.10B02固件包，我搞了半天才理解这句话的意思。因为是真•包含在里面。

![](1.png)

![](2.png)

下载中间版本就可以了。然后就下载DIR-878的固件，我这里需要的是DIR-878 v1.12的固件，这个版本是已经加密固件。



#### 提取DIR-882 v1.04B02固件

直接用binwalk解包就可以了,这里选择-Me参数，递归的提取文件，因为这个固件还有挺多层的，这里直接一步到位。

```
binwalk -Me firmware
```

在cpio-root目录下就可以看到整个文件系统。解密的文件imgdecrypt就在/bin目录下面。



#### 固件解密

这个地方要用QEMU执行跨架构的chroot，所以要下载qemu-mipsel-static组件，这个下载的话我直接`apt-get qemu-user-static`发现文件是空的，所以直接网上下载文件包。下载完成后把`qemu-mipsel-static`复制到`./usr/bin/`里面。这里要注意一定要放`qemu-mipsel-static`文件，也不要放错目录，如果抱文件格式错误或者没有文件或目录的错误就有可能是这个原因。

在cpio-root目录下，执行`sudo chroot . /bin/sh`就可以进入模拟的执行环境。接下来利用之前提到的imgdecrypt来解密文件了，把要解密的固件放在cpio-root目录下面即可。

![](3.png)

固件解密之后就可以用binwalk解包了。

![](4.png)



啊，这篇博客也太短了。



