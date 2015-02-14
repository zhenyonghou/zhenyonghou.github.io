# 重载全局new/delete实现内存检测
-----

下面介绍用重载new/delete运算符的方式来实现一个简单的内存泄露检测工具，基本思想是重载全局new/delete运算符，被检测代码调用new和delete运算符时就会调用重载过的operator new和operator delete，在重载的operator new里和operator delete里记录下内存申请和释放信息，从而判断内存使用情况。
下面一步步介绍它的实现！

##1. 全局new/delete的重载
先看一下重载new/delete的规则：
重载的operator new的参数个数任意，但第一个参数必须是size_t类型的，返回值必须是void*。重载operator delete只允许有一个参数，且是void*型。
当然，不光要重载operator new 和 operator delete, 还要重载operator new [], operator delete []，更多operator new和operator delete重载的内容参考《Effective C++》
重载的new/delete, new[]/delete[]代码如下：
```ruby
void * operator new (size_t size){
if(0 == size){
        return 0;
}
void *p = malloc(size);
return p;
}
 
void * operator new [](size_t size){
return operator new(size);
}
 
void operator delete (void * pointer){
if(0 != pointer){
free(pointer);
}
}
 
void operator delete[](void * pointer){
       operator delete(pointer);
}
```

##2. 用__FILE__, __LINE__记录new的位置
为了找到内存泄露的元凶，我要记录下每一处new所在的文件名和所在行。于是再次重载了operator new：
```
void * operator new (size_t size, const char* file, const size_t line);
void * operator new [](size_t size, const char* file, const size_t line);
```
这时候用VC6编译会看到警告：warning C4291(没有与operator new(unsigned int,const char *,const unsigned int) 匹配的delete)，又重载了
```
void operator delete (void * pointer, const char* file, const size_t line)；
void operator delete[](void * pointer, const char* file, const size_t line)；
```
尽管我知道它没用。

我想到了用系统提供的__FILE__和 __LINE__宏获取当前文件名与行号，我试图把__FILE__和 __LINE__直接填到operator new和operator new[]函数体里边，然后把函数置成inline，结果都输出的是重载operator new和operator new[]的文件和函数体printf函数所在行。然后又试了将operator new的缺省参数设为__FILE__和 __LINE__结果依然，于是想到了用宏定义。
先看看MFC里的做法，MFC为了调试方便，对new进行了宏定义：
```
#define new DEBUG_NEW
#define DEBUG_NEW new(THIS_FILE, __LINE__)
```
 
这里我借用MFC的做法，我也用宏定义：
```
void * operator new (size_t size, const char* file, const size_t line)；
void * operator new [](size_t size, const char* file, const size_t line)；
#define MC_NEW new(__FILE__, __LINE__)
#define new MC_NEW
```

##3. 将malloc/free 用new/delete替换
为了便于统计malloc/free信息，也用宏定义的方法处理：

```
#define malloc(s) ((void*)new unsigned char[s])
#define free(p)   (delete [] (char*)(p));
```
 
##4.在数据结构里存储内存使用情况。
下面写一个用于存储new/delete中内存信息的数据结构，可以使用链表，也可以使用哈希表，这里选用哈希表，写一个CHash类。
代码略。
 
##5. 为了保证CHash在所有对象析构执行完之后再销毁，应该将CHash放在全局存储区，将其设成static类型，另外，如果有多个static，还需要注意放置的顺序。

到这里这个简易的内存泄露检测工具完成了，很遗憾，还不能用于多线程。


再一次感谢您花费时间阅读这份欢迎稿，点击 <i class="icon-file"></i> (Ctrl+Alt+N) 开始撰写新的文稿吧！祝您在这里记录、阅读、分享愉快！

作者 [侯振永][1]     
写于2010 年 10月 18日 

[1]: https://zhenyonghou.github.io/