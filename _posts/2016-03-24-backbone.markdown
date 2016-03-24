---
layout: post
date:   2016-03-24 22:51
categories: blog
---

Events类在backbone库中扮演着很重要的角色，backbone的View与Model都用到了它。

目前项目中使用到的backbone是v1.1.2版，下面的代码分析也是基于此版本。也去backbone网站下载了最新的v1.3.2，代码复杂了些，基本的原理还是一样的。

Events提供的方法：

on,off,trigger,once,listenTo,stopListening,listenToOnce

这里重点介绍前面3个方法。


### on方法


------

作者 [侯振永][1]     
写于2016 年 3月 24日

[1]: https://zhenyonghou.github.io/