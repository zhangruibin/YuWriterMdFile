
# Java 集合系列02之 Collection架构
<div id="cnblogs_post_body" class="blogpost-body">

&nbsp;

### **<span style="font-family: 黑体; font-size: 18pt; line-height: 1.5; color: #000000;">概要</span>**

<span style="font-size: 16px;">首先，我们对Collection进行说明。下面先看看Collection的一些框架类的关系图：</span>

<span style="font-size: 16px;">[![](https://images0.cnblogs.com/blog/497634/201309/08172429-1ecddb7a87e347369ffc7c1c30f18396.jpg)](https://images0.cnblogs.com/blog/497634/201309/08172429-1ecddb7a87e347369ffc7c1c30f18396.jpg)</span>

<span style="font-size: 16px; color: #000000;">Collection是一个接口，它主要的两个分支是：**List** 和 **Set**。</span>

<span style="font-size: 16px; color: #000000;">List和Set都是接口，它们继承于Collection。**List是有序的队列，List中可以有重复的元素**；而**Set是数学概念中的集合，Set中没有重复元素**！</span>
<span style="font-size: 16px; color: #000000;">List和Set都有它们各自的实现类。</span>

<span style="color: #000000;">&nbsp;&nbsp;<span style="font-size: 16px; line-height: 1.5;">为了方便，我们抽象出了AbstractCollection抽象类，它实现了Collection中的绝大部分函数；这样，在Collection的实现类中，我们就可以通过继承AbstractCollection省去重复编码。AbstractList和AbstractSet都继承于AbstractCollection，具体的List实现类继承于AbstractList，而Set的实现类则继承于AbstractSet。</span></span>

<span style="font-size: 16px; color: #000000;">&nbsp; 另外，Collection中有一个iterator()函数，它的作用是返回一个Iterator接口。通常，我们通过Iterator迭代器来遍历集合。ListIterator是List接口所特有的，在List接口中，通过ListIterator()返回一个ListIterator对象。</span>

<span style="font-size: 16px; color: #000000;">&nbsp; 接下来，我们看看各个接口和抽象类的介绍；然后，再对实现类进行详细的了解。</span>

<span style="font-size: 16px;"><span style="line-height: 24px;">本章内容包括：
[1 Collection简介](#a1)
[2 List简介](#a2)
[3 Set简介](#a3)
[4 AbstractCollection](#a4)
[5 AbstractList](#a5)
[6 AbstractSet](#a6)
[7 Iterator](#a7)
[8 ListIterator](#a8)
</span></span>

<span style="font-size: 16px;"><span style="font-size: medium;"><span style="line-height: 24px;">转载请注明出处：[http://www.cnblogs.com/skywang12345/p/3308513.html](http://www.cnblogs.com/skywang12345/p/3308513.html)</span></span></span>

&nbsp;

### **<span style="font-family: 黑体; font-size: 18pt; line-height: 1.5; color: #000000;"><a name="a1"></a>1 Collection简介</span>**

<span style="font-size: 16px;">Collection的定义如下：</span>

<div class="cnblogs_code">
<pre><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">interface</span> Collection&lt;E&gt; <span style="color: #0000ff;">extends</span> Iterable&lt;E&gt; {}</pre>
</div>

<span style="font-size: 16px; color: #000000;">**它是一个接口，是高度抽象出来的集合，它包含了集合的基本操作：添加、删除、清空、遍历(读取)、是否为空、获取大小、是否保护某元素等等。**</span>

<span style="font-size: 16px; color: #000000;">  Collection接口的所有子类(直接子类和间接子类)都必须实现2种构造函数：不带参数的构造函数 和 参数为Collection的构造函数。带参数的构造函数，可以用来转换Collection的类型。</span>

<div class="cnblogs_code"><div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"></span></div>
<pre><span style="color: #008000;">//</span><span style="color: #008000;"> Collection的API
</span>

<span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         add(E object)
</span>
<span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         addAll(Collection&lt;? <span style="color: #0000ff;">extends</span> E&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">void</span><span style="color: #000000;">            clear()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         contains(Object object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         containsAll(Collection&lt;?&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         equals(Object object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">int</span><span style="color: #000000;">             hashCode()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         isEmpty()
</span><span style="color: #0000ff;">abstract</span> Iterator&lt;E&gt;<span style="color: #000000;">     iterator()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         remove(Object object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         removeAll(Collection&lt;?&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         retainAll(Collection&lt;?&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">int</span><span style="color: #000000;">             size()
</span><span style="color: #0000ff;">abstract</span> &lt;T&gt;<span style="color: #000000;"> T[]         toArray(T[] array)
</span><span style="color: #0000ff;">abstract</span> Object[]        toArray()</pre>
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"></span></div></div>

&nbsp;

### **<span style="font-family: 黑体; font-size: 18pt; line-height: 1.5; color: #000000;"><a name="a2"></a>2 List简介</span>**

<span style="font-size: 16px; line-height: 1.5;">List的定义如下：</span>

<div class="cnblogs_code">
<pre><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">interface</span> List&lt;E&gt; <span style="color: #0000ff;">extends</span> Collection&lt;E&gt; {}</pre>
</div>

<span style="color: #000000;">**<span style="font-size: 16px;"> List是一个继承于Collection的接口，即List是集合中的一种。List是有序的队列，List中的每一个元素都有一个索引；第一个元素的索引值是0，往后的元素的索引值依次+1。和Set不同，List中允许有重复的元素。</span>**</span>
<span style="font-size: 16px; color: #000000;">_  List的官方介绍如下_：</span>

<div class="cnblogs_code">
<pre>A List is a collection which maintains an ordering <span style="color: #0000ff;">for</span> its elements. Every element in the List has an index. Each element can thus be accessed by its index, with the first index being zero. Normally, Lists allow duplicate elements, as compared to Sets, where elements have to be unique.</pre>
</div>

&nbsp;

<span style="font-size: 16px;">关于API方面。既然List是继承于Collection接口，它自然就包含了Collection中的全部函数接口；由于List是有序队列，它也额外的有自己的API接口。主要有“添加、删除、获取、修改指定位置的元素”、“获取List中的子队列”等。</span><span style="font-size: 16px;">
</span>

<div class="cnblogs_code"><div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"></span>
</div>
<pre><span style="color: #008000;">//</span><span style="color: #008000;"> Collection的API</span>

<span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         add(E object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         addAll(Collection&lt;? <span style="color: #0000ff;">extends</span> E&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">void</span><span style="color: #000000;">            clear()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         contains(Object object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         containsAll(Collection&lt;?&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         equals(Object object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">int</span><span style="color: #000000;">             hashCode()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         isEmpty()
</span><span style="color: #0000ff;">abstract</span> Iterator&lt;E&gt;<span style="color: #000000;">     iterator()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         remove(Object object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         removeAll(Collection&lt;?&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         retainAll(Collection&lt;?&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">int</span><span style="color: #000000;">             size()
</span><span style="color: #0000ff;">abstract</span> &lt;T&gt;<span style="color: #000000;"> T[]         toArray(T[] array)
</span><span style="color: #0000ff;">abstract</span><span style="color: #000000;"> Object[]        toArray()
</span><span style="color: #008000;">//</span><span style="color: #008000;"> 相比与Collection，List新增的API：</span>
<span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">void</span>                add(<span style="color: #0000ff;">int</span><span style="color: #000000;"> location, E object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>             addAll(<span style="color: #0000ff;">int</span> location, Collection&lt;? <span style="color: #0000ff;">extends</span> E&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> E                   get(<span style="color: #0000ff;">int</span><span style="color: #000000;"> location)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">int</span><span style="color: #000000;">                 indexOf(Object object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">int</span><span style="color: #000000;">                 lastIndexOf(Object object)
</span><span style="color: #0000ff;">abstract</span> ListIterator&lt;E&gt;     listIterator(<span style="color: #0000ff;">int</span><span style="color: #000000;"> location)
</span><span style="color: #0000ff;">abstract</span> ListIterator&lt;E&gt;<span style="color: #000000;">     listIterator()
</span><span style="color: #0000ff;">abstract</span> E                   remove(<span style="color: #0000ff;">int</span><span style="color: #000000;"> location)
</span><span style="color: #0000ff;">abstract</span> E                   set(<span style="color: #0000ff;">int</span><span style="color: #000000;"> location, E object)
</span><span style="color: #0000ff;">abstract</span> List&lt;E&gt;             subList(<span style="color: #0000ff;">int</span> start, <span style="color: #0000ff;">int</span> end)</pre>
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy">
</span></div></div>

&nbsp;

### **<span style="font-family: 黑体; font-size: 18pt; line-height: 1.5; color: #000000;"><a name="a3"></a>3 Set简介</span>**

<span style="font-size: 16px;">Set的定义如下：</span>

<div class="cnblogs_code">
<pre><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">interface</span> Set&lt;E&gt; <span style="color: #0000ff;">extends</span> Collection&lt;E&gt; {}</pre>
</div>

<span style="color: #000000;">**<span style="font-size: 16px;"> Set是一个继承于Collection的接口，即Set也是集合中的一种。Set是没有重复元素的集合。</span>**</span>

<span style="font-size: 16px; color: #000000;">  关于API方面。Set的API和Collection完全一样。</span>

<div class="cnblogs_code"><div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy">
</span></div>
<pre><span style="color: #008000;">//</span><span style="color: #008000;"> Set的API</span>

<span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         add(E object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         addAll(Collection&lt;? <span style="color: #0000ff;">extends</span> E&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">void</span><span style="color: #000000;">             clear()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         contains(Object object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         containsAll(Collection&lt;?&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         equals(Object object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">int</span><span style="color: #000000;">             hashCode()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         isEmpty()
</span><span style="color: #0000ff;">abstract</span> Iterator&lt;E&gt;<span style="color: #000000;">     iterator()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;">         remove(Object object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         removeAll(Collection&lt;?&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span>         retainAll(Collection&lt;?&gt;<span style="color: #000000;"> collection)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">int</span><span style="color: #000000;">             size()
</span><span style="color: #0000ff;">abstract</span> &lt;T&gt;<span style="color: #000000;"> T[]         toArray(T[] array)
</span><span style="color: #0000ff;">abstract</span> Object[]         toArray()</pre>
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy">
</span></div></div>

&nbsp;

### **<span style="font-family: 黑体; font-size: 18pt; line-height: 1.5; color: #000000;"><a name="a4"></a>4 AbstractCollection</span>**

<span style="font-size: 16px;">AbstractCollection的定义如下：</span>

<div class="cnblogs_code">
<pre><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">class</span> AbstractCollection&lt;E&gt; <span style="color: #0000ff;">implements</span> Collection&lt;E&gt; {}</pre>
</div>

<span style="font-size: 16px;"> AbstractCollection是一个抽象类，它实现了Collection中除iterator()和size()之外的函数。</span>
<span style="font-size: 16px;">  AbstractCollection的主要作用：它实现了Collection接口中的大部分函数。从而方便其它类实现Collection，比如ArrayList、LinkedList等，它们这些类想要实现Collection接口，通过继承AbstractCollection就已经实现了大部分的接口了。</span>

&nbsp;

### **<span style="font-family: 黑体; font-size: 18pt; line-height: 1.5; color: #000000;"><a name="a5"></a>5 AbstractList</span>**

<span style="font-size: 16px;">AbstractList的定义如下：</span>

<div class="cnblogs_code">
<pre><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">class</span> AbstractList&lt;E&gt; <span style="color: #0000ff;">extends</span> AbstractCollection&lt;E&gt; <span style="color: #0000ff;">implements</span> List&lt;E&gt; {}</pre>
</div>

<span style="font-size: 16px;"> AbstractList是一个继承于AbstractCollection，并且实现List接口的抽象类。它实现了List中除size()、get(int location)之外的函数。</span>
<span style="font-size: 16px;">  AbstractList的主要作用：它实现了List接口中的大部分函数。从而方便其它类继承List。</span>
<span style="font-size: 16px;">  另外，和AbstractCollection相比，AbstractList抽象类中，实现了iterator()接口。</span>

&nbsp;

### **<span style="font-family: 黑体; font-size: 18pt; line-height: 1.5; color: #000000;"><a name="a6"></a>6 AbstractSet</span>**

<span style="font-size: 16px;">AbstractSet的定义如下：</span>&nbsp;

<div class="cnblogs_code">
<pre><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">class</span> AbstractSet&lt;E&gt; <span style="color: #0000ff;">extends</span> AbstractCollection&lt;E&gt; <span style="color: #0000ff;">implements</span> Set&lt;E&gt; {}</pre>
</div>

<span style="font-size: 16px;"> AbstractSet是一个继承于AbstractCollection，并且实现Set接口的抽象类。由于Set接口和Collection接口中的API完全一样，Set也就没有自己单独的API。和AbstractCollection一样，它实现了List中除iterator()和size()之外的函数。</span>
<span style="font-size: 16px;">  AbstractSet的主要作用：它实现了Set接口中的大部分函数。从而方便其它类实现Set接口。</span>

&nbsp;

### **<span style="font-family: 黑体; font-size: 18pt; line-height: 1.5; color: #000000;"><a name="a7"></a>7 Iterator</span>**

<span style="font-size: 16px;">Iterator的定义如下：</span>

<div class="cnblogs_code">
<pre><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">interface</span> Iterator&lt;E&gt; {}</pre>
</div>

<span style="font-size: 16px;"> Iterator是一个接口，它是集合的迭代器。集合可以通过Iterator去遍历集合中的元素。Iterator提供的API接口，包括：是否存在下一个元素、获取下一个元素、删除当前元素。</span>
<span style="font-size: 16px;">  注意：Iterator遍历Collection时，是fail-fast机制的。即，当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。关于fail-fast的详细内容，我们会在后面专门进行说明。TODO</span>

<div class="cnblogs_code">
<pre><span style="color: #008000;">//</span><span style="color: #008000;"> Iterator的API</span>
<span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;"> hasNext()
</span><span style="color: #0000ff;">abstract</span><span style="color: #000000;"> E next()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">void</span> remove()</pre>
</div>

&nbsp;

### **<span style="font-family: 黑体; font-size: 18pt; line-height: 1.5; color: #000000;"><a name="a8"></a>8 ListIterator</span>**

<span style="font-size: 16px;">ListIterator的定义如下：</span>

<div class="cnblogs_code">
<pre><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">interface</span> ListIterator&lt;E&gt; <span style="color: #0000ff;">extends</span> Iterator&lt;E&gt; {}</pre>
</div>

<span style="font-size: 16px;"> ListIterator是一个继承于Iterator的接口，它是队列迭代器。专门用于便利List，能提供向前/向后遍历。相比于Iterator，它新增了添加、是否存在上一个元素、获取上一个元素等等API接口。</span>

<div class="cnblogs_code"><div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy">
</span></div>
<pre>
<span style="color: #008000;">//</span><span style="color: #008000;"> ListIterator的API
</span>

<span style="color: #008000;">
//</span>
<span style="color: #008000;"> 继承于Iterator的接口</span>

<span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;"> hasNext()
</span><span style="color: #0000ff;">abstract</span><span style="color: #000000;"> E next()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">void</span><span style="color: #000000;"> remove()
</span><span style="color: #008000;">//</span><span style="color: #008000;"> 新增API接口</span>
<span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">void</span><span style="color: #000000;"> add(E object)
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">boolean</span><span style="color: #000000;"> hasPrevious()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">int</span><span style="color: #000000;"> nextIndex()
</span><span style="color: #0000ff;">abstract</span><span style="color: #000000;"> E previous()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">int</span><span style="color: #000000;"> previousIndex()
</span><span style="color: #0000ff;">abstract</span> <span style="color: #0000ff;">void</span> set(E object)</pre>
<div class="cnblogs_code_toolbar"><span class="cnblogs_code_copy"></span></div></div>

<span style="font-size: 16px; line-height: 1.5;">&nbsp;</span>
</div>