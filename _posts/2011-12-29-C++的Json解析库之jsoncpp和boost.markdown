---
layout: post
date:   2011-12-29 23:57
categories: c++
---

JSON(JavaScript Object Notation)跟xml一样也是一种数据交换格式，了解json请参考其官网http://json.org，本文不再对json做介绍，将重点介绍c++的json解析库的使用方法。json官网上列出了各种语言对应的json解析库，作者仅介绍自己使用过的两种C++的json解析库:jsoncpp(v0.5.0)和Boost(v1.34.0)。

###  一. 使用jsoncpp解析json
Jsoncpp是个跨平台的开源库，首先从http://jsoncpp.sourceforge.net/上下载jsoncpp库源码，我下载的是v0.5.0，压缩包大约107K，解压，在jsoncpp-src-0.5.0/makefiles/vs71目录里找到jsoncpp.sln，用VS2003及以上版本编译，默认生成静态链接库。 在工程中引用，只需要include/json及.lib文件即可。

 使用JsonCpp前先来熟悉几个主要的类： 

Json::Value     可以表示里所有的类型，比如int,string,object,array等，具体应用将会在后边示例中介绍。

Json::Reader   将json文件流或字符串解析到Json::Value, 主要函数有Parse。

Json::Writer    与Json::Reader相反，将Json::Value转化成字符串流，注意它的两个子类：Json::FastWriter和Json::StyleWriter，分别输出不带格式的json和带格式的json。

#### 1. 从字符串解析json

{% highlight cpp %}
int ParseJsonFromString()  
{  
  const char* str = "{\"uploadid\": \"UP000000\",\"code\": 100,\"msg\": \"\",\"files\": \"\"}";  
  
  Json::Reader reader;  
  Json::Value root;  
  if (reader.parse(str, root))  // reader将Json字符串解析到root，root将包含Json里所有子元素  
  {  
    std::string upload_id = root["uploadid"].asString();  // 访问节点，upload_id = "UP000000"  
    int code = root["code"].asInt();    // 访问节点，code = 100  
  }  
  return 0;  
}
{% endhighlight %}

#### 2. 从文件解析json

json文件内容：

{% highlight cpp %}
{  
    "uploadid": "UP000000",  
    "code": "0",  
    "msg": "",  
    "files":  
    [  
        {  
            "code": "0",  
            "msg": "",  
            "filename": "1D_16-35_1.jpg",  
            "filesize": "196690",  
            "width": "1024",  
            "height": "682",  
            "images":  
            [  
                {  
                    "url": "fmn061/20111118",  
                    "type": "large",  
                    "width": "720",  
                    "height": "479"  
                },  
                {  
                    "url": "fmn061/20111118",  
                    "type": "main",  
                    "width": "200",  
                    "height": "133"  
                }  
            ]  
        }  
    ]  
} 
{% endhighlight %}

解析代码：

{% highlight cpp %}
int ParseJsonFromFile(const char* filename)  
{  
  // 解析json用Json::Reader  
  Json::Reader reader;  
  // Json::Value是一种很重要的类型，可以代表任意类型。如int, string, object, array...  
  Json::Value root;         
  
  std::ifstream is;  
  is.open (filename, std::ios::binary );    
  if (reader.parse(is, root))  
  {  
    std::string code;  
    if (!root["files"].isNull())  // 访问节点，Access an object value by name, create a null member if it does not exist.  
      code = root["uploadid"].asString();  
      
    // 访问节点，Return the member named key if it exist, defaultValue otherwise.  
    code = root.get("uploadid", "null").asString();  
  
    // 得到"files"的数组个数  
    int file_size = root["files"].size();  
  
    // 遍历数组  
    for(int i = 0; i < file_size; ++i)  
    {  
      Json::Value val_image = root["files"][i]["images"];  
      int image_size = val_image.size();  
      for(int j = 0; j < image_size; ++j)  
      {  
        std::string type = val_image[j]["type"].asString();  
        std::string url = val_image[j]["url"].asString();  
      }  
    }  
  }  
  is.close();  
  return 0;  
}
{% endhighlight %}

#### 3. 在json结构中插入json

{% highlight cpp %}
Json::Value arrayObj;   // 构建对象  
Json::Value new_item, new_item1;  
new_item["date"] = "2011-12-28";  
new_item1["time"] = "22:30:36";  
arrayObj.append(new_item);  // 插入数组成员  
arrayObj.append(new_item1); // 插入数组成员  
int file_size = root["files"].size();  
for(int i = 0; i < file_size; ++i)  
  root["files"][i]["exifs"] = arrayObj;   // 插入原json中
{% endhighlight %}

#### 4. 输出json

{% highlight cpp %}
// 转换为字符串（带格式）  
std::string out = root.toStyledString();  
// 输出无格式json字符串  
Json::FastWriter writer;  
std::string out2 = writer.write(root); 
{% endhighlight %}

### 二. 使用Boost property_tree解析json
property_tree可以解析xml，json，ini，info等格式的数据，用property_tree解析这几种格式使用方法很相似。

解析json很简单，命名空间为boost::property_tree，reson_json函数将文件流、字符串解析到ptree，write_json将ptree输出为字符串或文件流。其余的都是对ptree的操作。

解析json需要加头文件：

{% highlight cpp %}
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
{% endhighlight %}

#### 1. 解析json

解析一段下面的数据：

{% highlight cpp %}
{  
  "code": 0,  
  "images":  
  [  
    {  
      "url": "fmn057/20111221/1130/head_kJoO_05d9000251de125c.jpg"  
    },  
    {  
      "url": "fmn057/20111221/1130/original_kJoO_05d9000251de125c.jpg"  
    }  
  ]  
}
{% endhighlight %} 

{% highlight cpp %}
int ParseJson()  
{  
  std::string str = "{\"code\":0,\"images\":[{\"url\":\"fmn057/20111221/1130/head_kJoO_05d9000251de125c.jpg\"},{\"url\":\"fmn057/20111221/1130/original_kJoO_05d9000251de125c.jpg\"}]}";  
  using namespace boost::property_tree;  
  
  std::stringstream ss(str);  
  ptree pt;  
  try{      
    read_json(ss, pt);  
  }  
  catch(ptree_error & e) {  
    return 1;   
  }  
  
  try{  
    int code = pt.get<int>("code");   // 得到"code"的value  
    ptree image_array = pt.get_child("images");  // get_child得到数组对象  
      
    // 遍历数组  
    BOOST_FOREACH(boost::property_tree::ptree::value_type &v, image_array)  
    {  
      std::stringstream s;  
      write_json(s, v.second);  
      std::string image_item = s.str();  
    }  
  }  
  catch (ptree_error & e)  
  {  
    return 2;  
  }  
  return 0;  
}
{% endhighlight %}

#### 2. 构造json

{% highlight cpp %}
int InsertJson()  
{  
  std::string str = "{\"code\":0,\"images\":[{\"url\":\"fmn057/20111221/1130/head_kJoO_05d9000251de125c.jpg\"},{\"url\":\"fmn057/20111221/1130/original_kJoO_05d9000251de125c.jpg\"}]}";  
  using namespace boost::property_tree;  
  
  std::stringstream ss(str);  
  ptree pt;  
  try{      
    read_json(ss, pt);  
  }  
  catch(ptree_error & e) {  
    return 1;   
  }  
  
  // 修改/增加一个key-value，key不存在则增加  
  pt.put("upid", "00001");  
  
  // 插入一个数组  
  ptree exif_array;  
  ptree array1, array2, array3;  
  array1.put("Make", "NIKON");  
  array2.put("DateTime", "2011:05:31 06:47:09");  
  array3.put("Software", "Ver.1.01");  
  exif_array.push_back(std::make_pair("", array1));  
  exif_array.push_back(std::make_pair("", array2));  
  exif_array.push_back(std::make_pair("", array3));  
  
//   exif_array.push_back(std::make_pair("Make", "NIKON"));  
//   exif_array.push_back(std::make_pair("DateTime", "2011:05:31 06:47:09"));  
//   exif_array.push_back(std::make_pair("Software", "Ver.1.01"));  
  
  pt.put_child("exifs", exif_array);  
  std::stringstream s2;  
  write_json(s2, pt);  
  std::string outstr = s2.str();  
  
  return 0;  
}  
{% endhighlight %}

### 三. 两种解析库的使用经验
1. 用boost::property_tree解析字符串遇到"\/"时解析失败，而jsoncpp可以解析成功，要知道'/'前面加一个'\'是JSON标准格式。

2. boost::property_tree的read_json和write_json在多线程中使用会引起崩溃。

针对1，可以在使用boost::property_tree解析前写个函数去掉"\/"中的'\'，针对2，在多线程中同步一下可以解决。

我的使用心得：使用boost::property_tree不仅可以解析json，还可以解析xml，info等格式的数据。对于解析json，使用boost::property_tree解析还可以忍受，但解析xml，由于遇到问题太多只能换其它库了。



作者 [侯振永][1]     
写于2011 年 12月 29日

[1]: https://zhenyonghou.github.io/