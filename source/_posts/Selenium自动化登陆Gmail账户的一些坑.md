---
title: Selenium自动化登陆Gmail账户的一些坑
date: 2022-05-22 20:29:45
tags: 爬虫
description: 宇宙尽头是爬虫...
---

太久没有写博客了，这几天写爬虫又遇到的了问题，找了一些资料，对解决问题有了一些思路，但是想起很久没有写博客了，想写个博客换个心情。因为太久没写，感觉都不知道怎么写了。

因为资格论文做邮箱安全自动化检测工具，想把检查邮件的过程也自动化了，想象是美好的，但是现实很残酷，做到现在感觉工程量真的非常的大。我最初预想在6月之前能初步完成，如果不能初步完成至少给一个比较好的结果。最近其实自己也一直在责怪自己，明明去年12月底就已经做了一些调研，可以开始做一些工作了，但是中途一是别的事情导致分心，二是觉得自己会有更好的idea，然后一直拖到了开学，拖到了三月份，但是中途事情依旧是超级多的，包括找实习、弄项目、写文档，但是我又是一个喜欢一件事情专心做完的人，所以真的非常操蛋。但是我觉得不能总是在后悔中度过，所以最近一段时间也算是一直在认真工作，缓解焦虑的情绪。

每天在工作的时候倒是还好，没有焦虑的时间，感觉比较充实，但是到睡前，想想未尽事宜就觉得很难受，不过最近也在想办法开解自己。

那么我目前在做的事情就是给各个邮箱写爬虫，主要有几个功能：登陆、查看收件箱or垃圾箱、获取UI显示的发件人、标记邮件、标记全部已读。

其实很简单，如果是简单的网站的话，可能写代码+调试完也就2h，顺利的话一天能写三个，但是Gmail，花了我1.5天时间，还是有点问题，然后周日晚上跑过来加班以抚慰内心的焦虑。

首先讲一下之前出现的一些问题：

##### 1.selenium click失效-1

元素定位没问题，明明元素也找到了，就是偏偏怎么也不能点。怎么调试都搞不定，因为一开始用的Safari的Driver，感觉Chrome还是好一点，虽然要求Driver版本和浏览器一致，最近Gmail也出现元素不能点击的问题了，提示错误是元素不可交互，然后动态调试的时候在console中执行button.click()没有一点问题，代码单步就偏偏出错，目前这个debug给更大的问题中断了。

**更新**

关于click失效的问题，我处理的是一个类似下拉菜单的东西，先要点击按钮，再点击具体操作，那么之前是：

```
menu = self.driver.find_element(By.XPATH, xpath)
menu.click()
time.sleep(3)
function = self.driver.find_element(By.XPATH, xpath)
function.cilck()
```

这里很确定我已经定位到元素，但是会出现`element not interactable`的报错，不能点击，但是在console里面，function.cilck()又可以点击成功。

先把鼠标移到元素附近，再进行点击：

```
menu = self.driver.find_element(By.XPATH, xpath)
menu.click()
function = self.driver.find_element(By.XPATH, xpath)
ActionChains(self.driver).move_to_element(function).perform()
function.click()
```

可以正常执行。

##### 2.邮箱使用iframe

QQ邮箱不同的区域使用不同的iframe，比如说边上的工具栏和收件箱页面，这就需要在爬取过程中不断的切换iframe，牢记两个功能:

```
EC.frame_to_be_available_and_switch_to_it()
driver.back()
```

在不同iframe中自由切换。

##### 3.HTML元素使用动态ID

这让元素变得比较不好定位，Gmail的一些元素就是动态生成的ID，这个通过XPATH来找的话，其实不会影响，然后还可以找一些其他的值，譬如title、data-tooltip之类的。

##### 4.selenium click失效-2

click失效还有一种可能，就是定位错了元素，可能是通过这种方法匹配出来的元素有多个，刚好点了错的。例如在Gmail中，如果点开过垃圾箱和收件箱，那么在收件箱中会匹配到垃圾箱中的元素，这里可能需要elements[0]、elements[1]来精准定位一下。

这里提供一个方法来避免一些不必要的调试过程，直接在浏览器的console里面调试CSS选择器或者XPATH：

```
$$(css.selector)
$x(xpath)
```

##### 5.reCAPTCHA反爬措施

因为一些频繁的登陆操作给账号弄些验证码之类的，真的非常无敌烦。这里提供几种思路：

（1）自动化验证过程，这个应该也有很多人做过，感觉应该有很多可以直接用的轮子；

（2）针对chrome的话，有一种是通过stack overflow来中转访问别的网站，这样子不需要验证，

参考链接：[**https://www.wangxiaofeng.site/python-selenium-google.html**](https://www.wangxiaofeng.site/python-selenium-google.html)

（3）添加cookie，这是我想到的第一个办法，也是准备尝试的，因为感觉是可实现并且也会适用于其他邮箱的；

发现网上相关的帖子也比较多，Chrome可以直接看cookie还是比较方便的，这里需要注意的是要找的cookie是“accounts.google.com”域名之下的，并不是你要访问的。之后就是选择要登录的用户，输入密码。

参考链接：https://stackoverflow.com/questions/53953343/using-selenium-to-login-to-gmail-save-cookies-and-then-use-requests

（4）将账户安全级别降低；

有看到一个这样的解决方法，不知道靠不靠谱。

https://myaccount.google.com/lesssecureapps 



不知道还会有什么奇葩的坑，想一想就很难受😢，~~先这样吧~~还是别了，这个我希望不会再加新的了。

