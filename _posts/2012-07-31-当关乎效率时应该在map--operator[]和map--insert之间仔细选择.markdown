---
layout: post
date:   2012-07-31 20:34
categories: c++
---

《Effective STL》条款24：当关乎效率时应该在map::operator[]和map::insert之间仔细选择.

如果你要更新已存在的map元素，operator[]更好，但如果你要增加一个新元素，insert则有优势.

更有效率的”添加或更新“函数（书中的函数我抠了出来~ ）

{% highlight cpp %}

template<typename MapType,   
        typename KeyArgType,   
        typename ValueArgType>  
typename MapType::iterator   
EfficientAddOrUpdate(MapType& m,   
                     const KeyArgType& k,   
                     const ValueArgType& v)  
{  
  typename MapType::iterator lb = m.lower_bound(k);  
  if(lb != m.end() && !(m.key_comp()(k, lb->first))) { // 它的键等价于k...  
    lb->second = v; // 更新这个pair的值  
    return lb;  
  }  
  else {  
    typedef typename MapType::value_type MVT;  
    return m.insert(lb, MVT(k, v)); // 为什么在insert方法中加入lb参数，请参照@3  
  }  
}  
{% endhighlight %}

当使用map::operator[]插入的元素不存在时，其效率很低，所以才有了上面的函数，它具体怎么个低法这里就不说了，看下面关键点：

#### 1 lower_bound
原型：iterator lower_bound ( const key_type& x );
返回一个iterator，指向容器中第一个不小于（大于或等于）x的元素。注意，map中的元素是按照key的顺序存储的。

#### 2 key_comp
key_comp是map的成员，c.key_comp()(x, y)仅当在c的排序顺序中x在y之前时返回真

#### 3 关于map::insert
给insert指定一个position参数，使其效率显著提升，这也是使用lower_bound查找而不用find查找的原因，find查找不到返回的iterator指向map::end。


作者 [侯振永][1]     
写于2012 年 7月 31日

[1]: https://zhenyonghou.github.io/