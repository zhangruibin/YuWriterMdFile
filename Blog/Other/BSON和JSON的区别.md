
# BSON和JSON的区别
## BSON

BSON是由10gen开发的一个数据格式，目前主要用于MongoDB中，是MongoDB的数据存储格式。BSON基于JSON格式，选择JSON进行改造的原因主要是JSON的通用性及JSON的schemaless的特性。
按照定义性来说：
BSON( Binary Serialized Document Format) 是一种二进制形式的存储格式，采用了类似于 C 语言结构体的名称、对表示方法，支持内嵌的文档对象和数组对象，具有轻量性、可遍历性、高效性的特点，可以有效描述非结构化数据和结构化数据。

BSON是一种类JSON的一种二进制形式的存储格式，简称Binary JSON，它和JSON一样，支持内嵌的文档对象和数组对象，但是BSON有JSON没有的一些数据类型，如Date和BinData类型。
BSON可以做为网络数据交换的一种存储形式，这个有点类似于Google的Protocol Buffer，但是BSON是一种schema-less的存储形式，它的优点是灵活性高，但它的缺点是空间利用率不是很理想.
总的来说：
BSON有三个特点：轻量性、可遍历性、高效性。
## JSON
JSON是 JavaScript Object Notation 的首字母缩写，单词的意思是javascript对象表示法，这里说的json指的是类似于javascript对象的一种数据格式。
使用JSON的格式与解析方便的可以表示一个对象信息，json有两种格式：①对象格式：{"key1":obj,"key2":obj,"key3":obj...}、②数组/集合格式：[obj,obj,obj...]。
## 1.更快的遍历速度

　　对json格式来说，太大的json结构会导致数据遍历非常慢。在json中，要跳过一个文档进行数据读取，需要对此文档进行扫描才行，需要进行麻烦的数据结构匹配，比如括号的匹配。 
　　而BSON对json的一大改进就是，它会将json的每一个元素的长度存在元素的头部，这样你只需要读取到元素长度就能直接seek到指定的点上进行读取了。

## 2.操作更简易

　　对JSON来说，数据存储是无类型的，比如你要修改基本一个值，从9到10，由于从一个字符变成了两个，所以可能其后面的所有内容都需要往后移一位才可以。 
　　而使用BSON，你可以指定这个列为数字列，那么无论数字从9长到10还是100，我们都只是在存储数字的那一位上进行修改，不会导致数据总长变大。 
　　当然，在mongoDB中，如果数字从整形增大到长整型，还是会导致数据总长变大的。

## 3.增加了额外的数据类型

　　JSON是一个很方便的数据交换格式，但是其类型比较有限。 
　　BSON在其基础上增加了“byte array”数据类型。这使得二进制的存储不再需要先base64转换后再存成JSON，大大减少了计算开销和数据大小。 
　　当然，在有的时候，BSON相对JSON来说也并没有空间上的优势，比如对{“field”:7}，在JSON的存储上7只使用了一个字节，而如果用BSON，那就是至少4个字节（32位）

　　目前在10gen的努力下，BSON已经有了针对多种语言的编码解码包。并且都是Apache 2 license下开源的。并且还在随着mongoDB进一步地发展。

## 综上所述：

**数据结构：** 
　　JSON是像字符串一样存储的，BSON是按结构存储的（像数组 或者说struct）

**存储空间** 
　　BSON>JSON

**操作速度** 
　　BSON>JSON。
　　比如，遍历查找：JSON需要扫字符串，而BSON可以直接定位

**修改：** 
　　JSON也要大动大移，bson就不需要。

# 数据结构简单对比
## BSON
### 简单document
```
{
    title:"MongoDB",
    last_editor:"192.168.1.122",
    last_modified:new Date("27/06/2011"),
    body:"MongoDB introduction",
    categories:["Database","NoSQL","BSON"],
    revieved:false
}

```
### 嵌套数据类型


### 嵌套型
```
{
    name:"zhrb",
    age:"25",
    address:{
        country:"china",
        city:"bj",
        code:100000
    } ,
    scores:[
        {"name":"english","grade:99},
        {"name":"chinese","grade:100}
    ]
}

```
## JSON
### 简单型JSON
```
{
    title:"JSON",
    last_editor:"192.168.1.122",
    revieved:false
}

```
### 数组型JSON

```
{ 
"sites":[
{"name":"RebornChang的http博客" , "url":"www.zhangruibin.com"  },
 {  "name":"zhangruibin" , "url":"www.zhangruibin.com"  }, 
 {  "name":"RebornChang的https博客" , "url":"https://www.zhangruibin.com"  } 
  ]  
}
```
