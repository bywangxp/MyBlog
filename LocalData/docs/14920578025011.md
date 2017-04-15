# Iterator和Enumeration比较

原文博客：http://www.cnblogs.com/skywang12345/p/3311275.html
![屏幕快照 2017-04-13 12.32.07](media/14920578025011/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-04-13%2012.32.07.png)

上面的代码是HashTable中Iterator中的代码，可以看出使用Enumration比Iterator遍历所读更快，因为Hashtable通过Enumration去实现，而且Iterator中添加了对fail-fast机制的支持，所以时间长

```
package java.util;

public interface Enumeration<E> {

    boolean hasMoreElements();

    E nextElement();
}
```
```
package java.util;

public interface Iterator<E> {
    boolean hasNext();

    E next();

    void remove();
}
```

通过以上接口声明可知：
(01) 函数接口不同
Enumeration只有2个函数接口。通过Enumeration，我们只能读取集合的数据，而不能对数据进行修改。
Iterator只有3个函数接口。Iterator除了能读取集合的数据之外，也能数据进行删除操作。

(02) Iterator支持fail-fast机制，而Enumeration不支持。Enumeration 是JDK 1.0添加的接口。使用到它的函数包括Vector、Hashtable等类，这些类都是JDK 1.0中加入的，Enumeration存在的目的就是为它们提供遍历接口。Enumeration本身并没有支持同步，而在Vector、Hashtable实现Enumeration时，添加了同步。
而Iterator 是JDK 1.2才添加的接口，它也是为了HashMap、ArrayList等集合提供遍历接口。Iterator是支持fail-fast机制的：当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。


