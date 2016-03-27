---
layout: post
date:   2016-03-23 22:51
categories: blog
---

Events类在backbone库中扮演着很重要的角色，backbone的View与Model都用到了它。

目前项目中使用到的backbone是v1.1.2版，下面的代码分析也是基于此版本。官网最新的v1.3.2看，加入了新方法，代码复杂了些，核心的东西还是一样的。

Events提供的方法：

on,off,trigger,once,listenTo,stopListening,listenToOnce

这里重点介绍前面3个方法。


### on方法

{% highlight javascript %}

on: function(name, callback, context) {
  if (!eventsApi(this, 'on', name, [callback, context]) || !callback) return this;
  this._events || (this._events = {});
  var events = this._events[name] || (this._events[name] = []);
  events.push({callback: callback, context: context, ctx: context || this});
  return this;
},

{% endhighlight %}

首先调用eventsApi方法。

我们看到内部维护着event map，名为_events。map的key是事件名，也就是传进来的参数name；map的value是一个数组，输入参数callback和context组成一个成员。

到这里，大概也就明白Events的实现机制了。


关于eventsApi方法，只有在name是object或是有空格分割的事件时才有用，留意一下它的牛逼写法：

{% highlight javascript %}

for (var key in name) {
  obj[action].apply(obj, [key, name[key]].concat(rest));
}
...

{% endhighlight %}



### off方法

{% highlight javascript %}

off: function(name, callback, context) {
  var retain, ev, events, names, i, l, j, k;
  if (!this._events || !eventsApi(this, 'off', name, [callback, context])) return this;
  if (!name && !callback && !context) {
    this._events = void 0;
    return this;
  }
  names = name ? [name] : _.keys(this._events);
  for (i = 0, l = names.length; i < l; i++) {
    name = names[i];
    if (events = this._events[name]) {
      this._events[name] = retain = [];
      if (callback || context) {
        for (j = 0, k = events.length; j < k; j++) {
          ev = events[j];
          if ((callback && callback !== ev.callback && callback !== ev.callback._callback) ||
              (context && context !== ev.context)) {
            retain.push(ev);
          }
        }
      }
      if (!retain.length) delete this._events[name];
    }
  }

  return this;
},

{% endhighlight %}

如果name, callback, context都是null，则销毁_events.

如果name存在，则遍历_events[name]对应的队列，队列中每个单元分别与输入参数callback和context比较，查找移除项：

a. 两项都相等才移除；

b. 如果参数传进来的callback和context其中之一为null，则与另一个比较，相等就移除。


### trigger方法

{% highlight javascript %}

trigger: function(name) {
  if (!this._events) return this;
  var args = slice.call(arguments, 1);
  if (!eventsApi(this, 'trigger', name, args)) return this;
  var events = this._events[name];
  var allEvents = this._events.all;
  if (events) triggerEvents(events, args);
  if (allEvents) triggerEvents(allEvents, arguments);
  return this;
},

{% endhighlight %}

从_events中取出callbacks，看trigger里的处理：

{% highlight javascript %}

var triggerEvents = function(events, args) {
  var ev, i = -1, l = events.length, a1 = args[0], a2 = args[1], a3 = args[2];
  switch (args.length) {
    case 0: while (++i < l) (ev = events[i]).callback.call(ev.ctx); return;
    case 1: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1); return;
    case 2: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1, a2); return;
    case 3: while (++i < l) (ev = events[i]).callback.call(ev.ctx, a1, a2, a3); return;
    default: while (++i < l) (ev = events[i]).callback.apply(ev.ctx, args); return;
  }
};

{% endhighlight %}

可以看到，队列里callback一一执行，需要注意的是，callback最多支持3个参数。



### 总结

通过代码了解到这些：

- Events内部维护着一个event map，event与callback是一对多的关系。
- callback执行顺序按照监听顺序。
- callback参数个数有限制，最多支持3个。

还有很重要的一点：上面的实现会导致循环引用！比如on/off方法没成对调用等情况。

一直认为javascript是引用计数来管理内存的。谨慎起见，还得再了解下javascript的内存管理方式。

在[这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)找到了介绍javascript管理内存的方法，里面讲了两点：

- 引用计数垃圾收集

- 标记-清除算法


了解到`标记-清除算法`是浏览器帮着做的，这下放心点了。

其它语言也有很多库用相似的方法实现了事件通知机制，C++和objective-c通过弱引用的方式解决循环引用问题，javascript不支持弱引用，hold不住的内存问题只能交给浏览器了。


------

作者 [侯振永][1]     
写于2016 年 3月 24日

[1]: https://zhenyonghou.github.io/