---
title: UEFI Handle与protocol
date: 2021-04-09 09:48:06
tags: UEFI
---

最近看了一些UEFI中handle和protocol的资料，这里做一个整理，写一点自己的理解。

<!--more-->

### 主要参考文章

> [UEFI Handle的来龙去脉](https://harmonyhu.com/2010/11/18/UEFI-Handle/)
>
> [UEFI 协议与句柄](https://blog.csdn.net/robinsongsog/article/details/98849049?spm=1001.2014.3001.5501)
>
> [如何理解UEFI中handle和protocol的概念](https://blog.csdn.net/lindahui2008/article/details/78796550)



### Handle,Protocol和Interface是什么

EDK II UEFI Driver Writer's Guide中对Handle和Protocol的描述如下：

> Handles are a collection of one or more protocols and protocols are data structures named by a GUID. The data structure for a protocol may contain data fields, services, both or none at all.

#### Handle

Handle由一个或多个Ptotocol组成，但是这个看起来好像很抽象的样子，Handle到底是一个什么东西呢？找到一些相关的描述：

> （1）每个uefi module都是一个image，而每个image对应都有一个ImageHandle，其实ImageHandle的类型就是EFI_HANDLE。
>
> （2）EFI_HANDLE是指向某种对象的指针，UEFI用它来表示某个对象。……当我们将一个.efi文件加载到内存中时，UEFI也会为该文件建立一个Image对象（此Image非图像的意思），这个Image对象也是一个EFI_HANDLE对象。在UEFI内部，EFI_HANDLE被理解为IHANDLE
>
> （3）比如定义一个变量EFI_HANDLE hExample，当你将它作为参数传递给service的时候，在service内部是这样使用它的：IHANDLE * Handle=(IHANDLE*)hExample。也就是说IHANDLE*才是handle的本来面目。为什么要弄的这么复杂呢？一是为了抽象以隐藏细节，二可能是为了安全。

并没有找到非常具体说Handle到底是什么的话，第一句话的原出处不知道在哪里了，第二句话是UEFI原理与编程里面的。第三句来自[UEFI Handle的来龙去脉](https://harmonyhu.com/2010/11/18/UEFI-Handle/)。

综合上面这几句描述，那么这里可以将handle简单理解为每个镜像文件都会有的一个句柄，用来指向这个镜像文件。镜像文件中可能会使用到很多外部的函数，而这个Handle的功能就是挂载镜像文件所需要的Protocol实例。

也有说“Handle其实就是一个不会重复的整型数字”，从IHandle的数据结构来看，也不是没有道理。

#### Protocol

感觉Protocol是什么都快要被讲烂了，Protocol本质是一个结构体，包含数据和服务(Service)，但是也可能是空的，但是每个Protocol都具有唯一的GUID，GUID相当于一个全局标识符，用于定位Protocol。

Protocol其实就相当于Library和libc的作用。

#### Interface

EDK II UEFI Driver Writer's Guide中对Interface的描述：

> The protocol is composed of a GUID and a protocol interface structure.
> Many times, the UEFI driver that produces a protocol interface maintains additional private data fields. The protocol interface structure itself simply contains pointers to the protocol function. 

Interface本质就是Protocol的实例，也很好理解，在使用Protocol的时候会有Install操作，这个操作就是生成*PROTOCOL_INTERFACE*结构体，再挂载到Handle上。一个Protocol可能具有很多不同的Interface，挂载在Handle上，所以其实Handle和Protocol并没有直接相连接。

![](image1.jpg)

### Handle,Protocol和Interface之间关系

[如何理解UEFI中handle和protocol的概念](https://blog.csdn.net/lindahui2008/article/details/78796550)中的这张图我觉得是目前画的最清晰的一张描述三者关系的图片。

原文提到

> 首先系统中有两个大的双向链表：
> 1.以gHandleList为表头的链表，链表中每个Entry为IHANDLE结构体，将系统中所有的handle链接到一起；
> 2.以mProtocolDatabase为表头的链表，链表中每个Entry为PROTOCOL_ENTRY结构体，用于将系统中所有的protocol链接到一起。
>
> 首先每个handle用一个IHANDLE结构体表示，IHANDLE中的Protocols指向一个双向链表（红色箭头表示），用于将所有install到该handle的所有protocol链接到一起，每一个install的protocol都由一个对应的PROTOCOL_INTERFACE结构体来表示。
> 其次，每个protocol由一个PROTOCOL_ENTRY来表示，一个protocol可能被install到多个handle中，所以每个HANDLE_ENTRY中的Protocols都由可能会由多个PROTOCOL_INTERFACE实例，并且这些PROTOCOL_INTERFACE实例也被链表链接起来（绿色箭头表示）。
> ————————————————
> 版权声明：本文为CSDN博主「河马虚拟化」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
> 原文链接：https://blog.csdn.net/lindahui2008/article/details/78796550

![](image2.jpeg)

```
///
/// PROTOCOL_INTERFACE - each protocol installed on a handle is tracked
/// with a protocol interface structure
///
typedef struct {
  UINTN                       Signature;
  /// Link on IHANDLE.Protocols
  LIST_ENTRY                  Link;   
  /// Back pointer
  IHANDLE                     *Handle;  
  /// Link on PROTOCOL_ENTRY.Protocols
  LIST_ENTRY                  ByProtocol; 
  /// The protocol ID
  PROTOCOL_ENTRY              *Protocol;  
  /// The interface value
  VOID                        *Interface; 
  /// OPEN_PROTOCOL_DATA list
  LIST_ENTRY                  OpenList;       
  UINTN                       OpenListCount;

} PROTOCOL_INTERFACE;
```

PROTOCOL_INTERFACE是Interface的结构体。

其中的Link的类型是EFI_LIST_ENTRY，这个结构体里面就是前向指针和后向指针，就是将它和Handle和其他Interface链接起来。

ByProtocol就是指向Protocol中的Protocols，这个Protocols就是一条连接Protocol实现的所有Interface的链，本质就是图中的绿线。

*Protocol就是指向对应Protocol的指针。

```
///
/// PROTOCOL_ENTRY - each different protocol has 1 entry in the protocol
/// database.  Each handler that supports this protocol is listed, along
/// with a list of registered notifies.
///
typedef struct {
  UINTN               Signature;
  /// Link Entry inserted to mProtocolDatabase
  LIST_ENTRY          AllEntries;  
  /// ID of the protocol
  EFI_GUID            ProtocolID;  
  /// All protocol interfaces
  LIST_ENTRY          Protocols;     
  /// Registerd notification handlers
  LIST_ENTRY          Notify;                 
} PROTOCOL_ENTRY;
```

PROTOCOL_ENTRY就是Protocol的结构体。

AllEntries用于是Protocol Database的连接，Protocols是用于连接所有Interface的。

```
///
/// IHANDLE - contains a list of protocol handles
///
typedef struct {
  UINTN               Signature;
  /// All handles list of IHANDLE
  LIST_ENTRY          AllHandles;
  /// List of PROTOCOL_INTERFACE's for this handle
  LIST_ENTRY          Protocols;      
  UINTN               LocateRequest;
  /// The Handle Database Key value when this handle was last created or modified
  UINT64              Key;
} IHANDLE;
```

IHANDLE的结构体除了Protocols和一个Key也的确没有其他东西。AllHandles就是用于连接HandleList，Protocols是连接所有interface，Key目前没有看到太多使用的地方。

以上这些结构体都可以在edk2/MdeMoudulePkg/Core/Dxe/Hand/Handle.h中找到。

edk2/MdeMoudulePkg/Core/Dxe/Hand/Handle.c中就是具体Install、Find的实现了。源代码阅读这个不知道还会不会写。



### 小结

Protocol就是UEFI中提供Library功能的函数库，每个Image（.efi文件）在运行过程中会需要用到一些外部函数，所以每个Image就有一个Handle来Install所有需要的Protocol，这些Protocol实例化之后就是Interface，挂载在Handle上。mProtocolDatabase就是一条包含系统中所有Protocol的双向链表，gHandleList就是包含系统中所有Handle的双向链表。

感觉目前分析Handle和Protocol分析的好的文章还蛮多的，我好像也没有什么产出，不过还是想记录一下自己这段时间看的东西。

