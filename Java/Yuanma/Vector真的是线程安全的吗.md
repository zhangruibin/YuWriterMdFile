 <blockquote>
转载请标明出处^_^<br> 原文首发于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">www.zhangruibin.com</a><br> 本文出自于：<a href="http://www.zhangruibin.com" rel="nofollow" target="_blank">RebornChang的博客</a>
</blockquote>

# Vector真的是线程安全的吗
 ```
 Vector类 继承了 AbstractList<E> 实现了 List<E>, RandomAccess, Cloneable, java.io.Serializable接口。
  AbstractList 类 继承了 AbstractCollection实现了 List<E>；
  AbstractCollection类 继承了 Collection<E>；
  Collection接口 继承了 Iterable接口；
 
 ```
 查看源码得知，所谓安全，是在方法上加上了synchronized关键字以保证单线程持有。
  但是，使用synchronized关键字做线程安全的话，效率低，且不可修改。因为synchronized关键字是JVM级别的，不可控，靠JVM内部机制进行关键字识别以及对象的持有和释放，因此，使用Vector的话，效率很慢。
 而且，使用了synchronized关键字，是可以达到局域性线程同步，但是并不是真正的线程安全。因为：单个方法的原子性（注：原子性，程序的原子性即不会被线程调度机制打断），并不能保证复合操作也具有原子性。
 即：复合操作有可能打破单方法的原子性。
 
 举个栗子：
 ```
 if (!vector.contains(element)) 
    vector.add(element); 
    then do sm
}
 ```
上面是经典的 put-if-absent 情况，尽管 contains, add 方法在vector源码中都使用synchronized进行同步了.
```
vector中的源码


public boolean contains(Object o) {
        return indexOf(o, 0) >= 0;
    }
      public synchronized int indexOf(Object o, int index) {
        if (o == null) {
            for (int i = index ; i < elementCount ; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index ; i < elementCount ; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
```
但作为 vector 之外的使用环境，仍然存在  race condition: 因为虽然条件判断 if (!vector.contains(element))与方法调用 vector.add(element);  都是原子性的操作 (atomic)，但在 if 条件判断为真后，那个用来访问vector.contains 方法的锁已经释放，在即将的 vector.add 方法调用 之间有间隙，在多线程环境中，完全有可能被其他线程获得 vector的 lock 并改变其状态, 此时当前线程的vector.add(element);  正在等待。只有当其他线程释放了 vector 的 lock 后，vector.add(element); 继续，但此时它已经基于一个错误的假设了。

因此：
单个的方法 synchronized 了并不代表组合（compound）的方法调用具有原子性，使 compound actions  成为线程安全的可能解决办法之一还是离不开intrinsic lock (这个锁应该是 vector 的，但由 client 维护)：
所以我们在使用的时候还要这样,在putIfAbsent的时候，先将vector对象进行加锁：
```
 public static void main(String[] args) {
        Vector vector = new Vector();
        vector.add("aaa");
        vector.add("666");
        putIfAbsent(vector,"666");
    }

    public static void putIfAbsent(Vector vector,Object object) {
        synchronized(vector) {
            if (!vector.contains(object)) {
                vector.add(object);
            }else {
                System.out.println("The vertor is contains the object of "+object.toString());
            }
        }
    }
    
    结果输出：
    The vertor is contains the object of 666
    
```
 结论：
 Vector 和 ArrayList 实现了同一接口 List, 但所有的 Vector 的方法都具有 synchronized 关键修饰。但对于复合操作，Vector 仍然需要进行同步处理。