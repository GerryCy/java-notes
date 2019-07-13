1、jvm，jre以及jdk三者之间的关系？java是如何实现跨平台的

2、序列化原理；类序列化时类的版本号的用途，如果没有指定一个版本号，系统是怎么处理的？如果加了字段会怎么样？哪些类有serialVersionUID属性，作用

3、int范围（基础数据类型） 基本数据类型，长度

4、字节与字符的区别

5、JDK动态代理与CGLIB动态代理原理

6、List接口和Set接口的区别

7、Collections的同步集合的包装
synchronizedMap 与ConcurrentHashMap区别；Collections.sort在jdk1.6以前是用的归并排序，1.7后变成了TimSort排序（归并优化）

8、finalize方法

9、final关键字用法

10、hashcode和equals区别，如果重写equals不重写hashcode会出现什么问题

11、equals与==区别

12、Error、Exception，Java异常体系，常见RuntimeException，受检异常和运行时异常的区别；受检异常的JVM实现原理

13、泛型；泛型擦除

14、String常量池，intern 

15、StringBuilder与StringBuffer

16、四种内部类

17、对象初始化顺序

18、try-finally-return

19、Integer的valueOf；为什么需要基础数据类型的包装类型

20、Iterator与ListIterator，Iterator实现原理

21、自动装箱、自动拆箱 int和Integer的区别

22、抽象类与接口 区别，使用场景 

23、static关键字：static修饰的变量并发下怎么保证变量的安全；出static修饰变量、修饰方法我会认为你合格，答出静态块，我会认为你不错，答出静态内部类我会认为你很好，答出静态导包我会对你很满意

24、ArrayList和LinkedList的使用场景

25、switch实现原理

26、HashMap的实现原理，为何按位与而不是取摸，hashmap的iterator读取时是否会读到另一个线程put的数据，红黑树；hashmap报ConcurrentModificationException的情况，手写HashMap的实现，set，get方法，不要求线程安全；hashmap，hashtable区别

27、hashset的实现原理 因为别人知道源码怎么实现的，故意构造相同的hash的字符串进行攻击，怎么处理 如果要去除后还要有序

28、面向对象的三个特性

29、arraylist linkedlist vector的区别

30、重写和重载

31、String有重写Object的hashcode和toString吗;String类添加功能，如何设计，可否继承

32、对象深拷贝与浅拷贝

33、Java的反射原理

34、队列 set map 区别

35、abstract 等关键字的作用，什么时候用

36、子类中如何调用父类的构造器，如果不用super关键字呢？有其他的方式吗？

37、java中所有类的父类是什么？他都有什么方法

38、解析XML的几种方式的原理与特点：DOM、SAX、PULL；Dom4J以及SAX的区别，什么时候用，怎么用

39、for each和正常for的用在不同数据结构（ArrayList、set、hashmap）上的效率区别

40、static class和non-static class的区别

41、String s=new String("abc")分别在堆栈上新建了哪些对象

42、Java 集合的快速失败机制 “fail-fast”

43、手写一个ArrayList类，实现add，remove，等基本的方法；对这个ArrayList进行改进，使之可以应对并发的场景

44、Collections.sort函数jdk7 和 jdk8 分别怎么实现的

45、为什么匿名内部类的变量必须用final修饰，编译器为什么要这么做，否则会出现什么问题；如何访问在其外面定义的变量？

46、PriorityQueue实现原理 替代：用TreeMap复杂度太高，有没有更好的方法。hash方法，但是队列不是定长的，如果改变了大小要rehash代价太大，还有什么方法？用堆实现，那每次get put复杂度是多少（lgN）

47、SimpleDateFormat 线程安全

48、Java代码编译过程

49、comparable接口和comparator接口实现比较的区别和用法

50、Arrays.sort

51、protected权限能否被包外访问；几种访问权限

52、linkedhashmap和hashmap，treemap的区别

53、BIO、NIO、AIO 重点NIO实现原理 NIO并不是严格意义上的非阻塞IO而应该属于多路复用IO；nio堆外内存，AIO在工程中如何实现的？（大概说了下ajax的回调函数），又问回调函数具体是怎么实现的（传递函数指针），NIO DirectMemory是否了解