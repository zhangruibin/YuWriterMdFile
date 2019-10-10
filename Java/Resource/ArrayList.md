 <blockquote>
转载请标明出处^_^<br> 原文首发于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">www.zhangruibin.com</a><br> 本文出自于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">RebornChang的博客</a>
</blockquote>

# ArrayList
 
 # ArrayList中的removeIf
 
  ArrayList 继承了 AbstractList，实现了 List<E>、 RandomAccess、 Cloneable、java.io.Serializable接口；
  
  AbstractList继承了  AbstractCollection，实现了 List接口；
  
  AbstractCollection实现了Collection接口；
  
  Collection接口继承了Iterable接口；
  
  Collection为顶级集合接口，实现了Iterable接口，可以迭代。
  
 
 
 Collction接口中在Jdk1.8之后新增了个removeIf，有啥用？
 removeIf源码@since 1.8：
 
 ```
 default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
 ```
 ArrayList对其进行重写：
 ```
 @Override
    public boolean removeIf(Predicate<? super E> filter) {
        //首先要验证规则不能为空
        Objects.requireNonNull(filter);
        // figure out which elements are to be removed
        // any exception thrown from the filter predicate at this stage
        // will leave the collection unmodified
        //计数器，记录revove掉的次数
        int removeCount = 0;
        //声明要remove掉的集合，声明为set，保证不重复remove
        final BitSet removeSet = new BitSet(size);
        final int expectedModCount = modCount;
        final int size = this.size;
        //循环遍历list
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            @SuppressWarnings("unchecked")
            final E element = (E) elementData[i];
            //boolean test(T t);校验参数是否匹配，匹配则返回true，如果匹配，则在removeSet中存入索引，removeCount值加一
            if (filter.test(element)) {
                removeSet.set(i);
                removeCount++;
            }
        }
        //校验此时的集合是否在多线程IO下受到改变，若受到改变则抛出异常
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        //源码中这句，表明了，将不需要移除的空间左移
        // shift surviving elements left over the spaces left by removed elements
        //定义anyToRemove，如果removeCount > 0则为true
        final boolean anyToRemove = removeCount > 0;
        //如果anyToRemove为true则继续进行，否则意味着没有匹配的可移除元素，直接return anyToRemove = false;
        if (anyToRemove) {
        //若有元素符合移除条件，则声明新size，重新定义list大小(ArrayList初始默认容量为10)
            final int newSize = size - removeCount;
            //循环遍历元素赋值，这里用到了一个新的方法nextClearBit(fromIndex)
            //将需要移除的元素在重新存储的时候跳过
            for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
            
                i = removeSet.nextClearBit(i);
                elementData[j] = elementData[i];
            }
            //如果新的集合与原集合比较，将指定要移除的设置为null,GC回收内存
            for (int k=newSize; k < size; k++) {
                elementData[k] = null;  // Let gc do its work
            }
            //更新全局变量的集合大小为移除后的新集合大小
            this.size = newSize;
            //再次判断是否受到多线程IO影响
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            modCount++;
        }

        return anyToRemove;
    }
    //官方解释：返回设置为在指定的起始索引上或之后出现的第一个位的索引。
     public int nextClearBit(int fromIndex) {
        // Neither spec nor implementation handle bitsets of maximal length.
        // See 4816253.
        if (fromIndex < 0)
            throw new IndexOutOfBoundsException("fromIndex < 0: " + fromIndex);

        checkInvariants();

        int u = wordIndex(fromIndex);
        if (u >= wordsInUse)
            return fromIndex;

        long word = ~words[u] & (WORD_MASK << fromIndex);

        while (true) {
            if (word != 0)
                return (u * BITS_PER_WORD) + Long.numberOfTrailingZeros(word);
            if (++u == wordsInUse)
                return wordsInUse * BITS_PER_WORD;
            word = ~words[u];
        }
    }
     private static int wordIndex(int bitIndex) {
        return bitIndex >> ADDRESS_BITS_PER_WORD;
    }

 ```
 
 //验证结果
  ```
 public static void main(String[] args) {
        List list = new ArrayList();
        list.add("Name");
        list.add("aaa");
        list.add("bbb");
        list.add("ccc");
        list.add("ddd");
        System.out.println(list.toString());
        //removeIf 的使用方法
        list.removeIf(next ->next.toString().indexOf("a")>0);
        System.out.println("===============removeIf 1====================");
        System.out.println(list.toString());
        //或者这样写
        Predicate<String> predicate = (s) -> s.equals("ddd");
        list.removeIf(predicate);
        System.out.println("===============removeIf 2====================");
        System.out.println(list.toString());
        List list2 = new LinkedList();
        list2.addAll(list);
        System.out.println("===================addAll================");
        System.out.println(list2.toString());
        List list3 = new LinkedList();
        list3.addAll(list);
        list3.clear();
        System.out.println("===================clear================");
        System.out.println(list3.toString());
        list3.addAll(list);
        list3.addAll(list);
        list3.removeAll(list2);
        System.out.println("===================removeAll 1================");
        System.out.println(list3.toString());
        List list4 = new LinkedList();
        list4.addAll(list);
        list2.add("test");
        System.out.println("===================removeAll 2================");
        System.out.println(list4.toString());
    }
    
    输出：
    [Name, aaa, bbb, ccc]
    ===============removeIf====================
    [aaa, bbb, ccc]
    ===================addAll================
    [aaa, bbb, ccc]
    ===================clear================
    []
    ===================removeAll 1================
    []
    ===================removeAll 2================
    [aaa, bbb, ccc]
  ```
  
  可以看到：
  
    
  1.removeIf方法是给定一个判断条件，符合条件即删除。
    
  2.clear和removeAll的区别是：clear是无条件清空，removeAll是移除两个集合的交集。
    