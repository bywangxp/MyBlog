# Arrays.copyof(···)与System.arraycopy(···)区别
1.首先观察先System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)的声明：该方法是用了native关键字，调用的为C++编写的底层函数，可见其为JDK中的底层函数。 
2.Arrays.copyOf();该方法对于不同的数据类型都有相应的方法重载.(针对复杂对象，基本对象)并且底层调用的是System.arraycopy();
1.copyOf()的实现是用的是arrayCopy(); 
2.arrayCopy()需要目标数组，对两个数组的内容进行可能不完全的合并操作。 
3.copyOf()在内部新建一个数组，调用arrayCopy()将original内容复制到copy中去，并且长度为newLength。返回copy; 



