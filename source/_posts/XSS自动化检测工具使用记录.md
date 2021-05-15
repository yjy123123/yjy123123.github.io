---
title: XSS自动化检测工具使用记录
date: 2021-05-15 14:03:04
tags: 工具使用
description: 记录XSpear和XssSniper两个工具的使用方法。
---



因为项目有相关的需求，所以就用了一下XSpear和XssSniper两个工具，记录一下使用方法。

### XSpear

项目地址：https://github.com/hahwul/XSpear

#### 安装

Readme里面每一个步骤都有，但是我当时还有有些地方不太清楚什么意思。而且感觉不是很有名的工具找他们的使用教程的话，基本上都是Readme翻译成中文就是一篇博客，毫无参考价值。

![](1.png)

这个添加我开始没太懂添加到哪里，有点傻眼。因为我真的不太知道gem、ruby这些，然后查了一下Gemfile。

> Gemfile 是我们创建的一个用于描述 gem 之间依赖的文件。gem 是一堆 Ruby 代码的集合，它能够为我们提供调用。你的 Gemfile 必须放在项目的根目录下面， 这是 Bundler 的要求，对于任何的其他形式的包管理文件来说，这也是标准。这里值得注意的一点是 Gemfile 会被作为 Ruby 代码来执行。当在 Bundler 上下文环境中被执行的时能使我们访问一些方法，我们用这些方法来解释 gem 之间的 require 关系。

或许这就是makefile？

不过查了一下就知道这是每个项目下面的一个文件，所以找到XSpear下载的位置，我的是在`/Library/Ruby/Gems/2.6.0/gems/XSpear-1.4.1`里面，这个在执行`gem install` 的时候可以看到具体下载的目录。

```
cd path
sudo vim Gefile
```

把`gem 'XSpear'`添加进去就可以了。

![](2.png)

#### XSpear on Burpsuite

接下来就是重点了，之所以选XSpear就是因为它能够配合Burpsuite使用，可以保持登陆会话信息。

这个工具的作者还专门在他的博客写了具体使用教程，比较详细，手把手教学，结合里面的截图，上手使用还是没有太大问题。

https://www.hahwul.com/2019/12/29/run-other-application-on-burp-suiteburp/

##### Step 1

在macos使用的话，需要用脚本把Burpsuite命令行转发到终端，这个脚本在项目包里面有，`/XSpear-1.4.1/forBurp/otwa.sh`，运行前记得提升一下脚本权限。

```
$ chmod 755 otwa.sh
```

##### step 2

在BApp Store里面下载Custom Send To插件，不用担心，社区版也可以用。Burpsuite里面Extender->BApp Store直接下载就可以了，Custom Send To就是一个可以扩展Send to菜单的插件，安装完之后可以看见toolbar里面就多了一个Send to的选项。

##### Step3

在Send to里面添加XSpear的相关配置。

```
--Entries
Name: XSpear
Command: xspear --raw %F -a -b {your-blind-xss-url}

--Miscellaneous
# MacOS 
~~your-path~~/open_terminal_with_args/otwa.sh %C
# Linux
(default) xterm %C
```

![](3.png)

##### step4

最后就是使用了，选中报文的全部内容->右键->Extensions->Sendto->Spear就可以了。

这里因为打马赛克好麻烦，mac自带的编辑图片的没办法打，就不贴图了。



### XssSniper

这个工具是Chrome的插件，使用非常的简单，用户手动浏览界面，这个工具会自动扫描，感觉也还蛮厉害的。

项目地址：

https://0kee.360.cn/domXss/

但是这个网址里面直接下载可能会失败，所以我用了[收藏猫插件](https://chrome.pictureknow.com/)下载。

因为这个工具的使用比较简单，就不在赘述了。



最后，因为五一放假，所以节后还是事情不少，感觉就没什么时间总结一下，一直都还蛮想找机会写博客记录一下的，感觉变化还是挺大的，从一开始硬想几个主题写博客，到现在自己主动想记录东西。养成这种总结的习惯真的还挺好的，几天看自己那篇mac技巧总结感觉自己都很多东西不记得了。