# LinkedList/Vector/CopyOnWriteArrayList 解析
##LinkedList
LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
LinkedList 实现 List 接口，能对它进行队列操作。
LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。
LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
LinkedList 是非同步的。

```

//定位具体的i值处的值
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```
LinkedList实际上是通过双向链表去实现的。既然是双向链表，那么它的顺序访问会非常高效，而随机访问效率比较低。
既然LinkedList是通过双向链表的，但是它也实现了List接口{也就是说，它实现了get(int location)、remove(int location)等“根据索引值来获取、删除节点的函数”}。LinkedList是如何实现List的这些接口的，如何将“双向链表和索引值联系起来的”？
>实际原理非常简单，它就是通过一个计数索引值来实现的。当index与长度的1/2比较，如果index大，则从last开始，否则从first开始。

 #jdk1.7中包含first last指针，是一个双向链表。而jdk1.6有一个header，header结点上不存储信息，第一个结点是header.next.element,最后一个结点是header.previous.element，是一个双向循环链表
 
 (01)包含内部类：Entry。Entry是双向链表节点所对应的数据结构，它包括的属性有：当前节点所包含的值，上一个节点，下一个节点。
(02) 从LinkedList的实现方式中可以发现，它不存在LinkedList容量不足的问题。
(03) LinkedList实现java.io.Serializable。当写入到输出流时，先写入“容量”，再依次写入“每一个节点保护的值”；当读出输入流时，先读取“容量”，再依次读取“每一个元素”。
(04) 由于LinkedList实现了Deque，而Deque接口定义了在双端队列两端访问元素的方法。提供插入、移除和检查元素的方法。每种方法都存在两种形式：一种形式在操作失败时抛出异常，另一种形式返回一个特殊值（null 或 false，具体取决于操作）。

### 对addAll函数的思考

　　在addAll函数中，传入一个集合参数和插入位置，然后将集合转化为数组，然后再遍历数组，挨个添加数组的元素，但是问题来了，为什么要先转化为数组再进行遍历，而不是直接遍历集合呢？从效果上两者是完全等价的，都可以达到遍历的效果。关于为什么要转化为数组的问题，我的思考如下：1. 如果直接遍历集合的话，那么在遍历过程中需要插入元素，在堆上分配内存空间，修改指针域，这个过程中就会一直占用着这个集合，考虑正确同步的话，其他线程只能一直等待。2. 如果转化为数组，只需要遍历集合，而遍历集合过程中不需要额外的操作，所以占用的时间相对是较短的，这样就利于其他线程尽快的使用这个集合。说白了，就是有利于提高多线程访问该集合的效率，尽可能短时间的阻塞。

##Vector
包含三个成员变量
(01) elementData 是"Object[]类型的数组"，它保存了添加到Vector中的元素。elementData是个动态数组，如果初始化Vector时，没指定动态数组的>大小，则使用默认大小10。随着Vector中元素的增加，Vector的容量也会动态增长，capacityIncrement是与容量增长相关的增长系数，具体的增长方式，请参考源码分析中的ensureCapacity()函数。

(02) elementCount 是动态数组的实际大小。

(03) capacityIncrement 是动态数组的增长系数。如果在创建Vector时，指定了capacityIncrement的大小；否则，每次当Vector中动态数组容量增加时>，增加的大小都是capacityIncrement。若容量增加系数 >0，则将容量的值增加“容量增加系数”；否则，将容量大小增加一倍。

##CopyOnWriteArrayList

```
    /** The lock protecting all mutators */
    final transient ReentrantLock lock = new ReentrantLock();

    /** The array, accessed only via getArray/setArray. */
    private transient volatile Object[] array;
```
 这两个成员变量是该类的核心，对集合所有的修改操作都需要使用lock加锁，array则是整个集合的数据储存部分，关键在于该array被声明为volatile，当一个线程对与线程中array副本的修改会立即同步到主内存中该变量中去。
 
```

public void add(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            if (index > len || index < 0)
                throw new IndexOutOfBoundsException("Index: "+index+
                                                    ", Size: "+len);
            Object[] newElements;
            int numMoved = len - index;
            if (numMoved == 0)
                newElements = Arrays.copyOf(elements, len + 1);
            else {
                newElements = new Object[len + 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index, newElements, index + 1,
                                 numMoved);
            }
            newElements[index] = element;
            setArray(newElements);
        } finally {
            lock.unlock();
        }
    }

```
在对数据进行插入之前，通过该lock.lock()方法对代码块加锁，通过比较index和length之间的位置，判断出需要移位的数目，最后通过System.arraycopy()方法，重新生产一个新的newElements数组，然后将该数组传递给array成员变量保存。



```
  public E get(int index) {
        return get(getArray(), index);
   }        
```
get方法则是取得该线程当前拥有的array数组，不需要额外的同步开销。
为什么get方法不需要同步呢？这正是CopyOnWriteArrayList的高效之处，在多线程环境下，每一个线程中都有一个主内存中变量的拷贝，该拷贝反映的是此时此刻主内存中该集合的情况。当线程开始运行时，一个线程会修改该集合，例如使用add方法，这个时候该线程内部的array变量就会修改，由于该变量是volatile的，所以此时该线程中修改的值会被立即同步到主内存中该变量中去。但是如果一个线程是在这个修改之前创建的，那么该线程内部所拥有的程序变量array还是改变之前的。所有对于CopyOnWriteArrayList类中的读取操作可能并不能真实的反映出此时此刻主内存块中的变量的情况，因此也不需要同步的开销。


##总结：
在CopyOnWriteArrayList里处理写操作（包括add、remove、set等）是先将原始的数据通过JDK1.6的Arrays.copyof()来生成一份新的数组
然后在新的数据对象上进行写，写完后再将原来的引用指向到当前这个数据对象（这里应用了常识1），这样保证了每次写都是在新的对象上（因为要保证写的一致性，这里要对各种写操作要加一把锁，JDK1.6在这里用了重入锁），
然后读的时候就是在引用的当前对象上进行读（包括get，iterator等），不存在加锁和阻塞，针对iterator使用了一个叫 COWIterator的阉割版迭代器，因为不支持写操作，当获取CopyOnWriteArrayList的迭代器时，是将迭代器里的数据引用指向当前引用指向的数据对象，无论未来发生什么写操作，都不会再更改迭代器里的数据对象引用，所以迭代器也很安全（这里应用了常识2）。
CopyOnWriteArrayList中写操作需要大面积复制数组，所以性能肯定很差，但是读操作因为操作的对象和写操作不是同一个对象，读之间也不需要加锁，读和写之间的同步处理只是在写完后通过一个简单的“=”将引用指向新的数组对象上来，这个几乎不需要时间，这样读操作就很快很安全，适合在多线程里使用，绝对不会发生ConcurrentModificationException ，所以最后得出结论：CopyOnWriteArrayList适合使用在读操作远远大于写操作的场景里，比如缓存。


