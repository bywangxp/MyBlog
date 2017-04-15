# HashMap源码解析 
参考http://www.cnblogs.com/skywang12345/p/3310835.html
##1.HashMap有四个构造方法，方法中有两个很重要的参数：初始容量和加载因子

>这两个参数是影响HashMap性能的重要参数，其中容量表示哈希表中槽的数量（即哈希数组的长度），初始容量是创建哈希表时的容量（默认为16），加载因子是哈希表当前key的数量和容量的比值，当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表提前进行 resize 操作（即扩容）。<font color = 'red'>如果加载因子越大，对空间的利用更充分，但是查找效率会降低（链表长度会越来越长）；如果加载因子太小，那么表中的数据将过于稀疏（很多空间还没用，就开始扩容了），严重浪费。</font>
JDK开发者规定的默认加载因子为0.75，因为这是一个比较理想的值。另外，无论指定初始容量为多少，构造方法都会将实际容量设为不小于指定容量的2的幂次方，且最大值不能超过2的30次方。

## get()
jdk1.6中get方法

```
 public V get(Object key) {
    if (key == null)
        return getForNullKey();
    // 获取key的hash值
    int hash = hash(key.hashCode());
    // 在“该hash值对应的链表”上查找“键值等于key”的元素
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
// 获取“key为null”的元素的值，HashMap将“key为null”的元素存储在table[0]位置，但不一定是该链表的第一个位置！
     private V getForNullKey() {
         for (Entry<K, V> e = table[0]; e != null; e = e.next) {
            if (e.key == null)
                 return e.value;
         }
         return null;
     } 
     
```
首先，如果key为null，则直接从哈希表的第一个位置table[0]对应的链表上查找。记住，key为null的键值对永远都放在以table[0]为头结点的链表中，当然不一定是存放在头结点table[0]中。如果key不为null，则先求的key的hash值，根据hash值找到在table中的索引，在该索引对应的单链表中查找是否有键值对的key与目标key相等，有就返回对应的value，没有则返回null

##put方法

```
// “key-value”添加到HashMap中
    public V put(K key, V value) {
        // 若“key为null”，则将该键值对添加到table[0]中。
        if (key == null)
            return putForNullKey(value);
        // 若“key不为null”，则计算该key的哈希值，然后将其添加到该哈希值对应的链表中。
        int hash = hash(key.hashCode());
        int i = indexFor(hash, table.length);
        for (Entry<K, V> e = table[i]; e != null; e = e.next) {
            Object k;
            // 若“该key”对应的键值对已经存在，则用新的value取代旧的value。然后退出！
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        // 若“该key”对应的键值对不存在，则将“key-value”添加到table中
        modCount++;
        // 将key-value添加到table[i]处
        addEntry(hash, key, value, i);
        return null;
    }
// 新增Entry。将“key-value”插入指定位置，bucketIndex是位置索引。
    void addEntry(int hash, K key, V value, int bucketIndex) {
        // 保存“bucketIndex”位置的值到“e”中
        Entry<K, V> e = table[bucketIndex];
        // 设置“bucketIndex”位置的元素为“新Entry”，
        // 设置“e”为“新Entry的下一个节点”
        table[bucketIndex] = new Entry<K, V>(hash, key, value, e);
        // 若HashMap的实际大小 不小于 “阈值”，则调整HashMap的大小
        if (size++ >= threshold)
            resize(2 * table.length);
    }
    
    void createEntry(int hash, K key, V value, int bucketIndex) {
    // 保存“bucketIndex”位置的值到“e”中
    Entry<K,V> e = table[bucketIndex];
    // 设置“bucketIndex”位置的元素为“新Entry”，
    // 设置“e”为“新Entry的下一个节点”
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    size++;
}
```
注意这里倒数第三行的构造方法，将key-value键值对赋给table[bucketIndex]，并将其next指向元素e，这便将key-value放到了头结点中，并将之前的头结点接在了它的后面。该方法也说明，每次put键值对的时候，<strong>总是将新的该键值对放在table[bucketIndex]处（即头结点处）</strong>。两外注意最后两行代码，每次加入键值对时，都要判断当前已用的槽的数目是否大于等于阀值（容量*加载因子），如果大于等于，则进行扩容，将容量扩为原来容量的2倍.
注意： createEntry() 一般用在 新增Entry不会导致“HashMap的实际容量”超过“阈值”的情况下。例如，我们调用HashMap“带有Map”的构造函数，它绘将Map的全部元素添加到HashMap中；但在添加之前，我们已经计算好“HashMap的容量和阈值”。也就是，可以确定“即使将Map中的全部元素添加到HashMap中，都不会超过HashMap的阈值”。

##分析求hash值和索引值的方法

```
   static int indexFor(int h, int length) {
        return h & (length-1);
     }
   static int hash(int h) {
            h ^= (h >>> 20) ^ (h >>> 12);
            return h ^ (h >>> 7) ^ (h >>> 4);
   }

     //确保返回表大小为2的整数次幂
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

首先，length为2的整数次幂的话，h&(length-1) 在数学上就相当于对length取模，这样便保证了散列的均匀，同时也提升了效率；
其次，length为2的整数次幂的话，则一定为偶数，那么 length-1 一定为奇数，奇数的二进制的最后一位是1，这样便保证了 h&(length-1) 的最后一位可能为0，也可能为1（这取决于h的值），即与后的结果可能为偶数，也可能为奇数，这样便可以保证散列的均匀，而如果length为奇数的话，很明显 length-1 为偶数，它的最后一位是0，这样 h&(length-1) 的最后一位肯定为0，即只能为偶数，这样导致了任何hash值都只会被散列到数组的偶数下标位置上，浪费了一半的空间，因此length取2的整数次幂，是为了使不同hash值发生碰撞的概率较小，这样就能使元素在哈希表中均匀地散列。

##面试问题
##HashMap的扩容机制，loaderFactor与initalCapciry

这是有两种策略，一种是不改变Capacity，因为即使桶占满了，我们还是可以利用每个桶附带的链表增加元素。但是这有个缺点，此时HaspMap就退化成为了LinkedList，使get和put方法的时间开销上升，这是就要采用另一种方法：增加Hash桶的数量，这样get和put的时间开销又回退到近于常数复杂度上。Hashmap就是采用的该方法
注意：initialCapacity，不宜取过大，因为实际的程序可能不仅仅使用get和put方法，也有可能使用迭代器，如initialCapacity容量较大，那么会使迭代器效率降低。所以理想的情况还是在使用HashMap前估计一下数据量。

##HashMap的key是如何散列到hash表的？相比较HashTable有什么改进？

我们一般对哈希表的散列很自然地会想到用hash值对length取模（即除留余数法），HashTable就是这样实现的，这种方法基本能保证元素在哈希表中散列的比较均匀，但取模会用到除法运算，效率很低，且hashtable直接使用了hashcode值，没有重新计算。
int index = (hash & 0x7FFFFFFF) % tab.length;
因为调用系统的产生的hashcode值可能为负数。

HashMap中则通过 h&(length-1) 的方法来代替取模，其中h是key的hash值，同样实现了均匀的散列，但效率要高很多，这也是HashMap对Hashtable的一个改进。

##与HashTable与HashMap的区别


1.HashTable比较古老， 是JDK1.0就引入的类，而HashMap 是 1.2 引进的 Map 的一个实现。HashTable 是线程安全的，能用于多线程环境中。Hashtable同样也实现了Serializable接口，支持序列化，也实现了Cloneable接口，能被克隆。由于HashMap非线程安全，在只有一个线程访问的情况下，效率要高于HashTable

2.Hashtable继承于Dictionary类，实现了Map接口。Dictionary是声明了操作"键值对"函数接口的抽象类。 有一点注意，HashTable除了线程安全之外（其实是直接在方法上增加了synchronized关键字，比较古老，落后，低效的同步方式），还有就是它的key、value都不为null。
另外Hashtable 也有 初始容量 和 加载因子。

    public Hashtable() {
        this(11, 0.75f);
    }
默认加载因子也是 0.75，HashTable在不指定容量的情况下的默认容量为11，而HashMap为16，Hashtable不要求底层数组的容量一定要为2的整数次幂，而HashMap则要求一定为2的整数次幂。因为HashTable是直接使用除留余数法定位地址。且Hashtable计算hash值，直接用key的hashCode()。

3.前面说了Hashtable中key和value都不允许为null，而HashMap中key和value都允许为null（key只能有一个为null，而value则可以有多个为null）。但NullPointerException异常，这是JDK的规范规定的。
4.Hashtable和HashMap扩容的方法不一样，HashTable中hash数组默认大小11，HashTable扩容方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数，增加为原来的2倍，没有加1。

##jdk1.8，HashMap新特性
对hashmap做了优化，引入红黑树。具体原理就是当hash表中每个桶附带的链表长度默认超过8时，链表就转换为红黑树结构，提高HashMap的性能，因为红黑树的增删改是O(logn)，而不是O(n)。
>static final int TREEIFY_THRESHOLD = 8;

```  
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0) // 首先判断hash表是否是空的，如果空，则resize扩容
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null) // 通过key计算得到hash表下标，如果下标处为null，就新建链表头结点，在方法最后插入即可
            tab[i] = newNode(hash, key, value, null);
        else { // 如果下标处已经存在节点，则进入到这里
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k)))) // 先看hash表该处的头结点是否和key一样（hashcode和equals比较），一样就更新
                e = p;
            else if (p instanceof TreeNode) // hash表头结点和key不一样，则判断节点是不是红黑树，是红黑树就按照红黑树处理
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else { // 如果不是红黑树，则按照之前的hashmap原理处理
                for (int binCount = 0; ; ++binCount) { // 遍历链表
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st (原jdk注释) 显然当链表长度大于等于7的时候，也就是说大于8的话，就转化为红黑树结构，针对红黑树进行插入（logn复杂度）
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k)))) 
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold) // 如果超过容量，即扩容
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```










