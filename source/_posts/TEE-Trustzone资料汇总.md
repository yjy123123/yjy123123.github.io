---
title: TEE&Trustzone资料汇总
date: 2021-01-31 19:46:11
tags: 底软
---



因为现有的参考资料比较多，内容也很多，所以这一篇不考虑展开写了。

简单概括两者关系：TEE（Trusted Execution Environment）就是一个可信执行环境，TrustZone提供的一种硬件解决方案实现TEE。
<!--more-->

TrustZone是ARM中的，Intel中的是SGX。TrustZone在移动端里面用的比较多。



## TrustZone参考资料

- [TrustZone官方文档](https://developer.arm.com/documentation)

  读官方文档永远是最靠谱最系统的

- [ARM Security Technology Building a Secure System using TrustZone Technology](https://blog.csdn.net/embededman/article/details/105768058)

  这篇博客是TrustZone白皮书的翻译

- [Trusted Execution Environments and Arm TrustZone](https://azeria-labs.com/trusted-execution-environments-tee-and-trustzone/)

  讲述TEE和TrustZone比较系统的博客，思路很清晰，推荐阅读，博主说后续会出一篇TEE攻击面相关的博客，期待一下。



## 其他TrustZone相关博客

- [简述ARM TrustZone](https://zhuanlan.zhihu.com/p/101486119)

  想要快速了解TrustZone是什么的一篇非常好的博客，对TrustZone架构、安全隐患和相关应用都进行了讲述，简单易懂，科普向，要深入了解还是不够。

- [introduction-to-trusted-execution-environment-arms-trustzone](https://blog.quarkslab.com/introduction-to-trusted-execution-environment-arms-trustzone.html)

  对TrustZone的介绍，但是有提到攻击面相关内容

- [一篇了解TrustZone](https://blog.csdn.net/guyongqiangx/article/details/78020257)

  基于白皮书对TrustZone总结的博客，如果不想读英文资料可以考虑看着一篇，比较精简



## TrustZone攻击面相关资料

- 论文：SoK: Understanding the Prevailing Security Vulnerabilities in TrustZone-assisted TEE Systems

  2020 S&P

  这篇论文系统化的总结了目前基于TrustZone的TEE相关的攻击面，非常的全面，并且附有相关漏洞实例，个人觉得读完这篇论文基本差不多可以知道TrustZone的攻击面了。

- 论文：VoltJockey: Breaching TrustZone by Software-Controlled Voltage Manipulation over

  清华2019年CCS上的一篇论文，通过软件来控制电压来进行攻击，这篇论文只看了一下摘要没有细看。

- [ARM TrustZone漏洞回顾](https://maxul.github.io/2020/09/27/TZ-revisit/)

  这篇博客是第一篇论文的阅读笔记
  

最后，好歹完成了一个月两篇的指标，虽然是31号冲刺一下，但是之前一直有思考应该写什么内容，备忘录里的东西挺零碎的，现在再拼凑起来好难啊，选了近期比较系统了解过的东西写了写。

不过好歹是迈出了第一步，虽然是有人督促（我简直就是抽一下动一下的驴子），但是希望自己能自觉坚持吧。坚持学习，坚持总结，坚持输出。

