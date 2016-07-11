---
layout: post
date:   2016-07-10 23:00
categories: blog
---

如今市面上的App绝大多数都是Tabbar形式的，进入App之后首先看到Tabbar管理着几个模块，当push进入下一个页面时Tabbar被覆盖，我近几年开发过的App也基本上都是这样的。
开发这样的App首先得解决好Tabbar和Navigation的关系，如果开始这个没处理好，后面处理跳转关系时将会痛苦不堪（我接手别人的项目时有过亲身体验，一种解铃还须系铃人的感觉）。在iOS上我早就有了比较优雅的解决方案。
现在开发React Native应用也得首先解决掉它，解决一次，以后开发其它App时就可以照搬了。


### Navigation的选择：

ReactNative库里提供了Navigation相关的几个组件：
Navigator、NavigatorIOS、NavigationExperimental，这么多，选哪个呀？ 首先因为要跨平台原因NavigatorIOS被排除，然后是NavigationExperimental还没正式上官方文档，排除。看了Navigator的使用方法，就选择了Navigator。

还遇到个NavigationBar的问题，按照在iOS上的使用经验，NavigationBar是公用的，Navigator正好也提供了NavigationBar，于是根据在iOS上的解决方案，使用了Navigator的NavigationBar，发现跳转关系维护起来太麻烦，原因是在iOS上苹果已经为我们解决好了一些问题，而Web版得自己解决。经过查看其它开源项目的代码，发现每个页面上放置一个NavigationBar控制比较方便，但是过渡效果肯定不是原生的那样了。最后还是自己写了个NavigationBar，贴到每个需要的页面上。

### Tab方案库的选择：

由于官方只有TabBarIOS，原因是Android上没有TabBar所以官方没提供。于是找到了react-native-tab-navigator

https://github.com/exponentjs/react-native-tab-navigator

它是iOS和Android平台通用的TabControl解决方案，问题解决得很到位，一般的开发用它就足够了。对于初学者，看看它源码收获也挺多。

### Tab与Navigator的结合：

现实是，TabBar管理着3个页面，要让push进来新的子页面覆盖住TabBar。

问题来了，应该是Navigator管理TabBar还是TabBar管理3个Navigator？

后者显然是达不到目的的，Navigator必须要管理TabBar，而且Navigator必须作为参数传递给TabBar的子页面，这样才能达到App里使用的是同一个Navigator。

解决方法考虑清楚了，再动手就不难了。

看代码（为突出重点，略去了一些代码）：

{% highlight javascript %}

export default class App extends Component {
  constructor(props) {
    super(props);
  }

  getInitialRoute() {
    return {
      component: AppTabContainer
    };
  }

  ...
  
  render() {
    return (
      <Navigator 
        initialRoute={this.getInitialRoute()}
        configureScene={this.configureScene}
        renderScene={this.renderScene}
      />
    );
  }
}

{% endhighlight %}


{% highlight javascript %}

export default class AppTabContainer extends Component {
  constructor(props) {
    super(props);
    this.state = {
      selectedTab: 'home'
    };
  }

  render() {
    return (
      <TabNavigator>
        <TabNavigator.Item
          ...
          <HomePage navigator={this.props.navigator}/>
        </TabNavigator.Item>
        <TabNavigator.Item
          ...
          <ProductPage navigator={this.props.navigator}/>
        </TabNavigator.Item>
        <TabNavigator.Item
          
          <ProfilePage navigator={this.props.navigator}/>
        </TabNavigator.Item>
      </TabNavigator>
    );
  }
}

{% endhighlight %}

Demo的源代码在这里：
https://github.com/zhenyonghou/TabNavi.git

------

作者 [侯振永][1]     
写于2016 年 7月 11日

[1]: https://zhenyonghou.github.io/