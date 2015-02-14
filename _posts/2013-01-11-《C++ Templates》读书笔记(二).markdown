---
layout: post
date:   2013-01-11 16:29
categories: c++
---

1. 模板实例化是在编译阶段进行。
2. 模板函数可以不指定类型，如max(3, 5)，模板类需要指定类型，如vector<int> my_vec;
3. 一个非模板函数可以和一个同名的模板函数同时存在，编译器会检查是否有实例化后相同的函数，如果有，则不实例化该实例。
4. 模板的函数实现要在头文件中，是因为扩展了inline函数吗？

模板类：

{% highlight cpp %}
template <typename T>
class Stack{
private:
  std::vector<T> elems; // 存储元素的容器
public：
  Stack();
  Stack(const Stack<T>& ); // 拷贝构造
  Stack<T>& operator= (const Stack<T>&);
  void push(const T& );
  T top() const;
};
{% endhighlight %}

这个类的类型是Stack<T>，其中T是模板参数.

当使用类名时必须使用Stack，比如类的构造、析构函数。

成员函数的实现：

1. 需要指定这是个模板函数，即template <typename T>

2. 需要类的完整类型限定符，即Static<T>::

实现如下：

{% highlight cpp %}
template <typename T>
void Static<T>::push(const T& node)
{
  elems.push_back(node);
}
{% endhighlight %}



作者 [侯振永][1]     
写于2013 年 1月 11日

[1]: https://zhenyonghou.github.io/