---
layout: post
date:   2012-07-28 22:07
categories: c++
---

### 第二章  函数模版

模板函数代表一个函数家族，看起来跟普通函数很相似，唯一区别就是模板函数中某些元素是未确定的，在使用时候这些元素被参数化。

返回两个值中最大者的模板函数：

{% highlight cpp %}
template<typename T>  
inline T const& max (T const& a, T const& b)  
{  
  return a < b ? b : a;  
}  
{% endhighlight %}

这个模板定义了一个“返回两者中最大者”的函数家族，输入两个参数，参数类型未确定，指定模板参数T来替代。
这个例子中，T需要支持operator<，因为两个参数是使用这个操作符进行比较的。
typename可以用class来替代，在这里两者等价，这是历史原因造成的，因为typename出现比较晚，先前被使用的class被保留了下来。
调用示例：

{% highlight cpp %}
int main()  
{  
  string s1 = "abcd", s2 = "ABCD";  
  string rs = ::max<string>(s1, s2);  
  
  int n1 = 10, n2 = 11;  
  int rn = ::max<int>(n1, n2);  
  
  double f1 = 7.8, f2 = 7.5;  
  double rf = ::max<double>(f1, f2);  
  
  return 0;  
}  
{% endhighlight %}

上面max被调用了3次，3次调用参数类型不同，实际上，max被实例化了3次，分别实例化成参数类型为string,int,double的函数。
只要使用函数模板，编译器会自动实例化，实例化发生在编译期间。所以如果参数使用不当，编译时就会报错。
### 第三章 类模版

#### 1. 类模板的定义

类模板的声明格式：


{% highlight cpp %}
template <typename T>  
class Stack{  
  ...  
}  
{% endhighlight %}

在类模板内部，T可以像其他任何类型一样使用，看下面的例子：

{% highlight cpp %}
template <typename T>  
class Stack{  
private:  
  std::vector<T> elemes;    // 存储元素的容器  
  
public:  
  Stack();                  // 构造函数  
  void  Push(const T& e);   // 压入元素  
  void  Pop();              // 弹出元素  
  T     Top() const;        // 返回栈顶元素  
};  
{% endhighlight %}

这个类的类型是Stack<T>，其中T是模板参数，因此当在声明中需要使用类的类型时，必须使用Stack<T>，例如要增加拷贝构造函数和赋值函数，就要这样写：

{% highlight cpp %}
template <typename T>  
class Stack{  
  ...  
  Stack(const Stack<T> &);  
  Stack<T>& operator=(const Stack<T> &);  
}  
{% endhighlight %}

然而，当使用类名而不是类的类型时，就应该只用Stack，譬如，当指定类名、构造函数、析构函数时就用Stack.
成员函数的实现：
1. 必须指定是一个模版函数，即加template <typename T>
2. 要使用类的完整限定符，即Stack<T>::

{% highlight cpp %}
template <typename T>  
void Stack<T>::Push(const T& e)  
{  
  elemes.push_back(e);  
}  
{% endhighlight %}

#### 2. 类模板的使用

{% highlight cpp %}
Stack<int> intStack;  
Stack<std::string> strStack;  
  
intStack.Push(7);  
strStack.Push("hello"); 
{% endhighlight %}

对于类模板，成员函数只有被调用的时候才被实例化，另外，如果类中含有静态成员，那么用来实例化的每种类型，都会实例化静态成员。
#### 3. 类模板的特化

类模板的特化跟函数重载类似，通过特化类模板，可以优化基于某种特定类型的实现。

要特化类模板，就要特化类模板的所有成员函数。
为了特化一个类模板，要在起始处声明一个template<>.接下来声明用来特化模版的类型，这个类型被用作模版实参。

{% highlight cpp %}
template<>  
class Stack<std::string>{  
  ...  
}  
{% endhighlight %}

进行类模板的特化时，每个成员都必须重新定义成普通函数，原来模版函数中每个T也相应地被特化的类型取代。

{% highlight cpp %}
void Stack<std::string>::Push(const std::string& e)  
{  
  elems.push_back(e);  
}  
{% endhighlight %}

#### 4. 局部特化

{% highlight cpp %}
template<typename T1, typename T2>  
class MyClass{  
  ...  
};  
{% endhighlight %}

上面的类模板有一下几种特化类型：

{% highlight cpp %}
// 局部特化，两个模板参数类型相同  
template<typename T>  
class MyClass<T, T>{  
  ...  
};  
  
// 局部特化，第2个模板参数是int  
template<typename T>  
class MyClass<T, int>{  
  ...  
};  
  
// 局部特化，两个模板参数都是指针类型  
template<typename T1, typename T2>  
class MyClass<T1*, T2*>{  
  ...  
};  
{% endhighlight %}

下面例子展示各种声明会使用哪个模板

{% highlight cpp %}
MyClass<int, float> mif;        // 使用 MyClass<T1, T2>  
MyClass<float, float> mff;      // 使用 MyClass<T, T>  
MyClass<float, int> mif;        // 使用 MyClass<T, int>  
MyClass<int*, float*> mif;      // 使用MyClass<T1*, T2*>  
{% endhighlight %}

#### 5.  缺省模版实参

可以为类模板的参数定义缺省值，这些值还可以引用之前的模板参数。
类模板参数有缺省值的例子：

{% highlight cpp %}
// 类模板参数有缺省值的例子：  
template <typename T, typename CONT = std::vector<T>>  
class Stack{  
private:  
  CONT elems;  
  
public:  
  void    Push(const T&);  
  void    Pop();  
  T       Top() const;  
};  
  
template <typename T, typename CONT>  
void Stack<T, CONT>::Push(const T& e)  
{  
  elems.push_back(e);  
}  
  
// 其它实现省略 ...  

{% endhighlight %}

使用示例：

{% highlight cpp %}
Stack<int> intStack;  // int栈  
// 存储double元素，使用deque管理double元素  
Stack<double, std::deque<double>> dblStack;  

{% endhighlight %}

### 第五章 技巧性基础知识

#### 1. typename关键字

当依赖于模板参数的名称是类型时，就应该使用typename.

{% highlight cpp %}
template<T>  
class MyClass  
{  
  typename T::SubType * ptr;  // typename用来说明SubType是定义于T内的一种类型，如果不使用typename，SubType会被认为是T的一个静态成员  
  ...  
};  
{% endhighlight %}

#### 2. 使用this->

在类模板中，在基类中声明的函数或变量，在调用时应该使用this->或Base<T>::限定访问，否则容易出错。

#### 3. 模板类型的变量的初始化

{% highlight cpp %}
template<typename T>  
void foo()  
{  
  T x = T();    // 如果T是内建类型，x是0或false.  
}  
  
template<typename T>  
class MyClass{  
private:  
  T x;  
public:  
  MyClass() : x() { // 通过初始化类表来初始化类模板成员  
  
  }  
};  
{% endhighlight %}



作者 [侯振永][1]     
写于2012 年 7月 28日

[1]: https://zhenyonghou.github.io/