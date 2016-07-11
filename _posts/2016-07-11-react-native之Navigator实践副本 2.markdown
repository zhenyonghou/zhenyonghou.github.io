---
layout: post
date:   2016-07-11 22:58
categories: blog
---

### 一. Navigator的配置

先来介绍下Navigator几个重要的props:

initialRoute: PropTypes.object

从哪里开始路由的，指定第一个scene，（这里不理解可以先继续往下看）


renderScene: PropTypes.func.isRequired,

render一个scene，

调用的参数：route, navigator，

返回一个scene：


{% highlight javascript %}
(route, navigator) =>
   <MySceneComponent title={route.title} navigator={navigator} />
{% endhighlight %}


configureScene: PropTypes.func

调用的参数：route, routeStack

返回一个scene configuration object

{% highlight javascript %}
(route, routeStack) => Navigator.SceneConfigs.FloatFromRight
{% endhighlight %}


可用的scene configuration object：
- Navigator.SceneConfigs.PushFromRight (default)
- Navigator.SceneConfigs.FloatFromRight
- Navigator.SceneConfigs.FloatFromLeft
- Navigator.SceneConfigs.FloatFromBottom
- Navigator.SceneConfigs.FloatFromBottomAndroid
- Navigator.SceneConfigs.FadeAndroid
- Navigator.SceneConfigs.HorizontalSwipeJump
- Navigator.SceneConfigs.HorizontalSwipeJumpFromRight
- Navigator.SceneConfigs.VerticalUpSwipeJump
- Navigator.SceneConfigs.VerticalDownSwipeJump



------

作者 [侯振永][1]     
写于2016 年 7月 11日

[1]: https://zhenyonghou.github.io/