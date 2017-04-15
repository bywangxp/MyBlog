# ArrayList源码解析
1.ArrayList 是一个数组队列，相当于 动态数组。

2.ArrayList包含了两个重要的对象：elementData 和 size。

(01) elementData 是"Object[]类型的数组"，它保存了添加到ArrayList中的元素。实际上，elementData是个动态数组，我们能通过构造函数 ArrayList(int initialCapacity)来执行它的初始容量为initialCapacity；如果通过不含参数的构造函数ArrayList()来创建ArrayList，则elementData的容量默认是10。elementData数组的大小会根据ArrayList容量的增长而动态的增长，具体的增长方式，请参考源码分析中的ensureCapacity()函数。

(02) size 则是动态数组的实际大小。
总结：
(01) ArrayList 实际上是通过一个数组去保存数据的。当我们构造ArrayList时；若使用默认构造函数，则ArrayList的默认容量大小是10。
(02) 当ArrayList容量不足以容纳全部元素时，ArrayList会重新设置容量：新的容量=“(原始容量x3)/2 + 1”。
 jdk1.8：newCapacity = oldCapacity + (oldCapacity >> 1);
(03) ArrayList的克隆函数，即是将全部元素克隆到一个数组中。
(04) ArrayList实现java.io.Serializable的方式。当写入到输出流时，先写入“容量”，再依次写入“每一个元素”；当读出输入流时，先读取“容量”，再依次读取“每一个元素”。
当我们调用ArrayList中的 toArray()，可能遇到过抛出“java.lang.ClassCastException”异常的情况。下面我们说说这是怎么回事。

(05)ArrayList提供了2个toArray()函数：

Object[] toArray()
<T> T[] toArray(T[] contents)
调用 toArray() 函数会抛出“java.lang.ClassCastException”异常，但是调用 toArray(T[] contents) 能正常返回 T[]。

toArray() 会抛出异常是因为 toArray() 返回的是 Object[] 数组，将 Object[] 转换为其它类型(如如，将Object[]转换为的Integer[])则会抛出“java.lang.ClassCastException”异常，因为Java不支持向下转型。具体的可以参考前面ArrayList.java的源码介绍部分的toArray()。
解决该问题的办法是调用 <T> T[] toArray(T[] contents) ， 而不是 Object[] toArray()。


