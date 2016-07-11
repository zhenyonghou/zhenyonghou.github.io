---
layout: post
date:   2016-07-10 22:58
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


一个简单的示例：

{% highlight javascript %}

export default class App extends Component {
  constructor(props) {
    super(props);
  }

  getInitialRoute() {
    return {
      component: FirstPage
    };
  }

  configureScene(route) {
    if (route.sceneConfig) {
      return route.sceneConfig;
    }
    return Navigator.SceneConfigs.PushFromRight;
  }

  renderScene(route, navigator) {
    let Component = route.component;
    return <Component {...route.params} navigator={navigator} />
  }

  render() {
    return (
      <Navigator
        initialRoute={this.getInitialRoute()}
        configureScene={this.configureScene}
        renderScene={this.renderScene}
        sceneStyle={{paddingTop: 0}}  /*64*/
      />
    );
  }
}

{% endhighlight %}


说明：

getInitialRoute方法将FirstPage作为首页。

configureScene

在每次转场的时候调用，指定了调用navigator.push()方法时如何转场，默认是PushFromRight。

该函数的前3行解决了个问题：

App中需要使用除了PushFromRight之外的其他转场动画时，可以通过sceneConfig参数指定。


进入下一级页面时候调用：

{% highlight javascript %}

this.props.navigator.push({
  component: NextPage,
});

{% endhighlight %}

NextPage页面就会从右向左push进来。

如果要让某个页面要从下往上present进来，可以这样：

{% highlight javascript %}

this.props.navigator.push({
  component: NextPage,
  sceneConfig: Navigator.SceneConfigs.FloatFromBottom,
});

{% endhighlight %}

renderScene

在每次转场的时候调用，route.component是一个页面，将params作为该页面的props传给该页面，另外还有个props是navigator，这样每个页面都可以通过this.props.navigator来操作路由了。
在这里，页面间参数传递问题解决了。比如传递参数id

{% highlight javascript %}

this.props.navigator.push({
  component: NextPage,
  params: {
    id: 100,
  }
});

{% endhighlight %}

在NextPage页面可以这样使用传过来的参数：

this.props.id


上面的代码将App的Navigator就算配置好了，可以用了，代码虽简单，把基本的问题都处理好了。


### 二. Navigator的使用

Navigator的使用很简单，不多说了，提供了几个很容易理解的方法：
- push()
- pop()
- ...

### 三. 遇到过的问题：

##### 1. NavigationBar的问题

Navigator提供了NavigationBar，但要在App里使用的话会遇到各种麻烦，建议还是自己写个NavigationBar，贴到每个页面的顶部。

关于NavigationBar上的返回箭头旁边的上一页名称，可以通过参数传递统一解决好。

##### 2. 使用不同的转场方式

上面已经提到了解决办法。

##### 3. 页面间参数如何传递

上面已经提到了解决办法。

##### 4. 转场时不要转场动画

未解决，如果你有解决方法，告诉我。


这里有一份代码可参考：
https://github.com/zhenyonghou/TabNavi.git

------

作者 [侯振永][1]     
写于2016 年 7月 11日

[1]: https://zhenyonghou.github.io/