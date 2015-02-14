---
layout: post
date:   2012-10-08 18:39
categories: c++
---

在boost的智能指针（包括scoped_ptr,scoped_array,shared_ptr,shared_array）里，会看到每个类都有一个成员函数（就称其为函数吧）operator unspecified_bool_type() const，而且是public的，如果你看过源码，也许会产生如何使用的困惑，下面贴出源码。

{% highlight cpp %}
typedef T * this_type::*unspecified_bool_type;  
  
operator unspecified_bool_type() const // never throws  
{  
  return ptr == 0? 0: &this_type::ptr;  
}  
{% endhighlight %}

当定义一个智能指针类的对象，用if语句判断对象是否为空指针的时候（先看下面的代码），也许还会有困惑。

{% highlight cpp %}

scoped_ptr<A> ptr(new A);  
  
if (ptr)  
{  
  // ...  
}  
{% endhighlight %}

至少我曾经有过这样的困惑，因为有不解，所以才去解读它。上面的两个困惑其实是一个问题（提前告诉你哈~），就是boost智能指针对bool语境的支持，智能指针之所以要支持bool语境，是为了在使用上接近原始指针。

先从下面这一句开始解读，

{% highlight cpp %}
typedef T * this_type::*unspecified_bool_type;  
{% endhighlight %}

这一句是声明了个指向this_type类成员变量的指针的类型，声明一个指向类成员变量的指针类型的格式：类型 类名::*指针类型，不明白可以参考下指向类成员函数的指针。
关于this_type，scoped_ptr代码中有typedef scoped_ptr<T> this_type;
通过这句代码得到的信息有：
1） unspecified_bool_type是个类型，而不是变量，由typedef得知。
2） unspecified_bool_type是scoped_ptr<T>类的成员变量的类型，该成员变量的类型是T *。

接下来分析这个函数：

{% highlight cpp %}
operator unspecified_bool_type() const // never throws  
{  
  return ptr == 0? 0: &this_type::ptr;  
}  

{% endhighlight %}

相信不少程序员都在这里卡住了，至少我是。原因是不知道operator关键字除了操作符重载外，还有另外一种用法，那就是隐式类型转换，没错，格式就是这样:

{% highlight cpp %}
operator type() {

...

}
{% endhighlight %}

执行if (ptr)时候便会执行operator unspecified_bool_type() const,转嫁成判断unspecified_bool_type类型的指针是否为空。


（完）


作者 [侯振永][1]     
写于2012 年 10月 8日

[1]: https://zhenyonghou.github.io/