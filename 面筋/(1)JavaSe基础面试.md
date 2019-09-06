## √1、锁相关

#### (1)线程安全:

​	当多个线程访问某个类的时候，不管运行环境采用何种调度方式或者这些线程如何交替执行，并且再主要代码中不需要进行任何额外的同步措施，这个类都能够表现出正确的执行行为，那么这个类就是线程安全的

#### (2)volatile

#### (3)synchronized

#### (4)synchronized和volatile的区别

#### (5)偏向锁

#### (6)轻量级锁

#### (7)自旋锁

#### (8)重量级锁

#### (9)锁粗化

#### (10)锁消除

#### (11)死锁

#### (12)如何预防死锁

#### (13)如果已经死锁了怎么解决

#### (14)不使用锁怎么实现线程安全（copyAndWrite)

   

## √2、jvm中对象的创建过程？jvm 里 new 对象时，堆会不会发生抢占？怎样设计jvm的堆的线程安全？

#### 	(1)对象创建

​	当 JVM 收到一个 new 指令时，会检查指令中的参数在**常量池**是否有这个类的符号引用，还会检查该类是否已经被加载过了，如果没有的话则要进行一次类加载。接着就是分配内存了，通常有两种方式：**指针碰撞和空闲列表**。

​	**指针碰撞**：使用指针碰撞的前提是堆内存是完全工整的，用过的内存和没用的内存各在一边每次分配的时候只需要将指针向空闲内存一方移动一段和内存大小相等区域即可。

​	**空闲列表**：当堆中已经使用的内存和未使用的内存互相交错时，指针碰撞的方式就行不通了，这时就需要采用空闲列表的方式。虚拟机会维护一个空闲的列表，用于记录哪些内存是可以进行分配的，分配时直接从可用内存中直接分配即可。

​	**注意点**：堆中的内存是否工整是有垃圾收集器来决定的，如果带有**压缩功能的垃圾收集器**就是采用指针碰撞的方式来进行内存分配的，比如Serial、ParNew等这些带有压缩功能的收集器。而比如CMS这种基于标记-交换方法实现的就需要采用空闲列表的方式。

#### 	(2)分配内存时也会出现并发问题:

​	**CAS**：在创建对象的时候使用 基于CAS 的乐观锁来保证。

​	**TLAB**：可以将内存分配安排在每个线程独有的空间进行，每个线程首先在堆内存中分配一小块内存，称为本地分配缓存(TLAB : Thread Local Allocation Buffer)。分配内存时，只需要在自己的分配缓存中分配即可，由于这个内存区域是线程私有的，所以不会出现并发问题。可以使用 -XX:+/-UseTLAB 参数来设定 JVM 是否开启 TLAB 。

#### 	(3)内存分配完毕

​	将分配到的内存空间都设置为零值，如果使用TLAB那么赋值为零值的过程可以提前至TLAB分配的时候进行。

​	内存分配之后需要对该对象进行设置，如对象头。对象头中右Mark Word字段、指向类元数据信息的类型指针（如果为数组，还有数组的长度信息）。对象头中的Mark Word字段存放这对象的哈希码、GC分代年龄、指向锁的指针、是否偏向锁（偏向锁ID、偏向时间戳）

​	最后执行类的<init>方法，生成一个可用的对象。

#### 	(4)对象访问

​	**直接指针方式**一个对象被创建之后自然是为了使用，在 Java 中是通过栈来引用堆内存中的对象来进行操作的。对于我们常用的 HotSpot 虚拟机来说，这样引用关系是通过**直接指针**来关联的。这样的好处就是：在 Java 里进行频繁的对象访问可以提升访问速度(相对于使用句柄池来说)。

​	**句柄**：堆中划分出一部分区域作为句柄池，在栈上的reference中存放的就是对象的句柄地址，在句柄中就包含了对象实例数据（堆）与类型数据（/static/方法区）的具体地址信息

#### 	(5)内存分配策略

​	**Eden 区分配**：简单的来说对象都是在堆内存中分配的，往细一点看则是优先在 Eden 区分配。这里就涉及到堆内存的划分了，为了方便垃圾回收，JVM 将堆内存分为新生代和老年代。而新生代中又会划分为 Eden 区，from Survivor、to Survivor 区。其中 Eden 和 Survivor 区的比例默认是 8:1:1，当然也支持参数调整 -XX:SurvivorRatio=8。当在 Eden 区分配内存不足时，则会发生 minorGC ，由于 Java 对象多数是朝生夕灭的特性，所以 minorGC 通常会比较频繁，效率也比较高。当发生 minorGC 时，JVM 会根据**复制算法**将存活的对象拷贝到另一个未使用的 Survivor 区，如果 Survivor 区内存不足时，则会使用**分配担保策略**将**对象移动到老年代中**。谈到 minorGC 时，就不得不提到 fullGC(majorGC) ，这是指发生在老年代的 GC ，不论是效率还是速度都比 minorGC 慢的多，回收时还会发生 stop the world 使程序发生停顿，所以应当尽量避免发生 fullGC 。

​	**老年代分配**：也有一些情况会导致对象直接在老年代分配，比如当分配一个大对象时(大的数组，很长的字符串)，由于 Eden 区没有足够大的连续空间来分配时，会导致提前触发一次 GC，所以尽量别频繁的创建大对象。因此 JVM 会根据一个阈值来判断大于该阈值对象直接分配到老年代，这样可以避免在新生代频繁的发生 GC。对于一些在新生代的老对象 JVM 也会根据某种机制移动到老年代中。JVM 是根据记录对象年龄的方式来判断该对象是否应该移动到老年代，根据新生代的复制算法，当一个对象被移动到 Survivor 区之后 JVM 就给该对象的年龄记为1，每当熬过一次 minorGC 后对象的年龄就 +1 ，直到达到阈值(默认为15)就移动到老年代中。可以使用 -XX:MaxTenuringThreshold=15 来配置这个阈值。

#### 	(6)如何获取堆和栈的dump文件？

​	JVM 的线程堆栈 dump 也称 core dump，内容为文本，主要包含当时 JVM 的线程堆栈信息，堆 dump 也称 heap dump，内容为二进制格式，主要包含当时 JVM 堆内存中的内容。由于各个操作系统、各个 JVM 实现不同，即使同一 JVM 实现，各个版本也有差异，下面描述的方法都基于 64 位 Linux 操作系统环境，Java 8 Oracle HotSpot JVM 实现

##### 	a)使用 jstack

​	jstack 是 JDK 自带的工具，用于 dump 指定进程 ID(PID)的 JVM 的线程堆栈信息。

```
打印堆栈信息到标准输出
jstack PID

打印堆栈信息到标准输出，会打印关于锁的信息
jstack -l PID

强制打印堆栈信息到标准输出，如果使用 jstack PID 没有响应的情况下(此时 JVM 进程可能挂起)，加 -F 参数
jstack -F PID
```

##### 	b)使用 jcmd

jcmd 是 JDK 自带的工具，用于向 JVM 进程发送命令，根据命令的不同，可以代替或部分代替 jstack、jmap 等。可以发送命令 `Thread.print` 来打印出 JVM 的线程堆栈信息。

```
# 下面的命令同等于 jstack PID
jcmd PID Thread.print

# 同等于 jstack -l PID
jcmd PID Thread.print -l
```

##### 	c)使用 kill -3

​	kill 可以向特定的进程发送信号(SIGNAL)，缺省情况是发送终止(TERM) 的信号 ，即 kill PID 与 kill -15 PID 或 kill -TERM PID 是等价的。JVM 进程会监听 QUIT 信号(其值为 3)，当收到这个信号时，会打印出当时的线程堆栈和堆内存使用概要，相比 jstack，此时多了堆内存的使用概要情况。但 jstack 可以指定 -l 参数，打印锁的信息。

```
kill -3 PID
# 或
kill -QUIT PID
```

##### 	d)jmap

​	jmap 也是 JDK 自带的工具，主要用于获取堆相关的信息。

##### 	e)堆 dump

```
# 将 JVM 的堆 dump 到指定文件，如果堆中对象较多，需要的时间会较长，子参数 format 只支持 b，即二进制格式
jmap -dump:format=b,file=FILE_WITH_PATH

# 如果 JVM 进程未响应命令，可以加上参数 -F 尝试
jmap -F -dump:format=b,file=FILE_WITH_PATH

# 可以只 dump 堆中的存活对象，加上 live 子参数，但使用 -F 时不支持 live
jmap -dump:live,format=b,file=FILE_WITH_PATH
```

## √3、java中是怎样自动装箱拆箱的

#### 	(1)装箱拆箱实现

​	装箱的时候自动调用的是包装类型的valueOf(xxx)方法。而在拆箱的时候自动调用的是包装类型的xxxValue方法。

#### 	(2)下面代码的输出

```java
public class Main {
    public static void main(String[] args) {
         
        Integer i1 = 100;
        Integer i2 = 100;
        Integer i3 = 200;
        Integer i4 = 200;
         
        System.out.println(i1==i2); //true
        System.out.println(i3==i4); //false
    }
}
```

下面是Integer的valueOf方法的源码

```java
public static Integer valueOf(int i) {
    if(i >= -128 && i <= IntegerCache.high)
        return IntegerCache.cache[i + 128];
    else
        return new Integer(i);
}
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;
        //创建的cache数组，大小为127-(-128)+1
        cache = new Integer[(high - low) + 1];
        //low为-128
        int j = low;
        //创建这个数组
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

​	valueOf方法创建Integer对象的时候，如果数值在[-128,127]之间，便返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象。上面的代码中i1和i2的数值为100，因此会直接从cache中取已经存在的对象，所以i1和i2指向的是同一个对象，而i3和i4则是分别指向不同的对象。

#### 	(3)注意点

- Integer、Short、Byte、Character、Long这几个类的valueOf方法的实现是类似的。Double、Float的valueOf方法的实现是类似的。

- Boolean的valueOf实现

  ```java
  public static final Boolean TRUE = new Boolean(true);
  public static final Boolean FALSE = new Boolean(false);
  //装箱
  public static Boolean valueOf(boolean b) {
      return (b ? TRUE : FALSE);
  }
  ```

- Double的valueOf实现

  ```java
  public static Double valueOf(double d) {
      return new Double(d);//返回的都是一个新的对象引用
  }
  ```

- Long的valueOf方法

  ```java
  public static Long valueOf(long l) {
      final int offset = 128;
      //在-128到127之间会右缓存
      if (l >= -128 && l <= 127) { // will cache
          return LongCache.cache[(int)l + offset];
      }
      return new Long(l);
  }
  ```

- 当 "=="运算符的两个操作数都是包装器类型的引用，则是比较指向的是否是同一个对象，而如果其中有一个操作数是表达式（即包含算术运算）则比较的是数值（即会触发自动拆箱的过程）。如同下面的例子

  ```java
  public static void main(String[] args) {
      Integer a = 1;
      Integer b = 2;
      Integer c = 3;
      Integer d = 3;
      Integer e = 321;
      Integer f = 321;
      Long g = 3L;
      Long h = 2L;
      System.out.println(c==d); //true
      System.out.println(e==f); //false
  
      //	true,包含算术运算，因此会触发自动拆箱过程（会调用intValue方法）因此它们比较的是数
      //值是否相等
      System.out.println(c==(a+b)); 
  
      //	true: c.equals(a+b)会先触发自动拆箱过程，再触发自动装箱过程，也就是说a+b，会先各
      //自调用intValue方法，得到了加法运算后的数值之后,便调用Integer.valueOf方法,再进行
      //equals比较
      System.out.println(c.equals(a+b));
      //	true: a+b先会出发自动拆箱过程，计算a+b的结果之直接和g=3进行‘==’的比较操作
      System.out.println(g==(a+b));
      //	fasle: a+b先会出发自动拆箱过程，计算a+b的结果返回的是该结果进行装箱(Integer.valueOf()方法)之后的引用，只用这个引用和g(Long类型的)进行比较，结果是false
      System.out.println(g.equals(a+b)); 
      //	true: (a+h)的返回的引用是Long类型的
      System.out.println(g.equals(a+h)); 
  }
  ```

  

- 谈谈Integer i = new Integer(xxx)和Integer i =xxx;这两种方式的区别。

  ```
  　　1）第一种方式不会触发自动装箱的过程；而第二种方式会触发；
  　　2）在执行效率和资源占用上的区别。第二种方式的执行效率和资源占用在一般性情况下要优于第一种情况（注意这并不是绝对的
  ```

  

## 4、Spring中的IOC怎样解决循环依赖问题



## √5、Java调度进程和线程



## √6、String，StringBuffer，StringBuilder区别以及使用场景

#### 	a)String

​		无论是sub操、concat还是replace操作都不是在原有的字符串上进行的，而是重新生成了一个新的字符串对象。也就是说进行这些操作后，最原始的字符串并没有被改变。

```java
# replace方法
return new String(buf, true);
# concat方法
return new String(buf, true);
# substring 方法
return ((beginIndex == 0) && (endIndex == count)) ? this :
        new String(offset + beginIndex, endIndex - beginIndex, value);
```

https://www.cnblogs.com/dolphin0520/p/3778589.html

#### √7、线程池、线程池的拒绝策略？线程池工作原理




11. ## beanFactory 和 FactoryBean 有啥区别；

12. ## spring IOC，解释一下，举个例子，如果不使用IOC用你的方式实现怎么实现；spring aop；

13. ## hashmap底层实现，自定义一个对象，插入过程（什么使用hasdcode，什么时候使用equals.....)；hashmap扩容多大、hash碰撞解决办法

14. ## concurrentHashMap说一下喽，1.8之后改进为啥用红黑树，不使用其他树。。。。。（数据过多，链表扩展成数组，即添加红黑树。查找方便，主要是红黑树维持着一种相对平衡没有AVL那么频繁，代价不会那么大）  

15. ## 不用线程的集合工具类的锁，如何保证hashMap的安全。。（想问自己加锁机制，你如何实现）

16. ## equals和==区别，为啥重写equals一定要重写hashcode

17. ## 字符串拼接操作、不用String的加号

18. ## SpringMVC中容器的bean能访问到Spring容器中的bean吗？反过来呢；事务传播

19. ## 线程wait了，被notifyall了，变成啥状态了，线程wait了会不会释放锁

20. ## mybatis的#好处

21. ## threadlocal，说说（结合源码）

22. ## 多态(为什么用多态)、多态的底层的原理

23. ## spring框架、依赖注入、spring注解

24. ## synchronized锁住对象和.class区别，synchronized原理、对象头除了存线程ID，还能存什么内容

25. ## jvm垃圾回收 很详细  内存模型

26. ## redis底层数据结构

27. ## Servlet相关接口、Servlet里面有哪些方法、doGet, doPost区别

28. ## mybits一二级缓存

29. ## NIO BIO AIO的区别

30. ## redis持久化区别

31. ## 类加载机制双亲委派和打破双亲委派、对象生命周期、自己实现类加载器

32. ## Java集合了解哪些？说了list set map的特点，每个中常用的类型，ArrayList与linklist的区别，查询与插入的时间复杂度，

33. ## 线程的创建方式，callable与runnable的区别

34. ## Java内存模型、JVM内存机制，垃圾回收机制、 gc 垃圾收集器知道哪些、java虚拟机回收算法。回收机制。回收器，年轻代到达老年代（讲了很细）。回收过程，jvm中什么样的可以被回收。对象从进入堆区到死亡的全流程

35. ## 工厂模式

36. ## 新生代老年代为什么这么划分

37. ## 单例模式 代理模式 springAOP、模版模式，动态代理模式、静态代理、观察者模式

38. ## CountDownLatch、CyclicBarrier区别、场景题目

39. ## Docker讲一下，和虚拟机比为啥好，为啥非用docker

40. ## java实现多线程消费者和生产者

41. ## 虚拟机运行时数据区。其中有什么数据。哪些区域是共享的。是私有的。

42. ## AQS、Lock原理、CLH同步队列怎么实现非公平与公平的？

43. ## 一个请求在 SpringMVC 中的流程

44. ## Java反射，泛型的原理，怎么实现的

45. ## Java垃圾回收算法有几种，分别是什么?G1垃圾收集器过程及特点? CMS垃圾收集器采用哪种垃圾回收算法?CMS垃圾收集器垃圾回收具体过程?CMS垃圾收集器有几次Stop the Word，具体是什么情况发生?

46. ## CAS是什么，优缺点？

47. ## spring中autowired resource区别

48. ## Redis的基本数据结构，Set怎么实现的，跳表的插入删除查询时间复杂度

49. ## 一致性hash，作用是什么

50. ## 负载均衡算法有哪些

51. ## jvm，jre以及jdk三者之间的关系？java是如何实现跨平台的

52. ## 序列化原理；类序列化时类的版本号的用途，如果没有指定一个版本号，系统是怎么处理的？如果加了字段会怎么样？哪些类有serialVersionUID属性，作用

53. ## int范围（基础数据类型） 基本数据类型，长度

54. ## 字节与字符的区别

55. ## JDK动态代理与CGLIB动态代理原理

56. ## List接口和Set接口的区别

57. ## Collections的同步集合的包装synchronizedMap 与ConcurrentHashMap区别；Collections.sort在jdk1.6以前是用的归并排序，1.7后变成了TimSort排序（归并优化）

58. ## finalize方法

59. ## final关键字用法

60. ## hashcode和equals区别，如果重写equals不重写hashcode会出现什么问题

61. ## equals与==区别

62. ## Error、Exception，Java异常体系，常见RuntimeException，受检异常和运行时异常的区别；受检异常的JVM实现原理

63. ## 泛型；泛型擦除

64. ## String常量池，intern 

65. ## 四种内部类

67. ## 对象初始化顺序

68. ## try-finally-return

69. ## Iterator与ListIterator，Iterator实现原理

71. ## 抽象类与接口 区别，使用场景 

73. ## static关键字：static修饰的变量并发下怎么保证变量的安全；出static修饰变量、修饰方法我会认为你合格，答出静态块，我会认为你不错，答出静态内部类我会认为你很好，答出静态导包我会对你很满意

74. ## ArrayList和LinkedList的使用场景

75. ## switch实现原理

76. ## HashMap的实现原理，为何按位与而不是取摸，hashmap的iterator读取时是否会读到另一个线程put的数据，红黑树；hashmap报ConcurrentModificationException的情况，手写HashMap的实现，set，get方法，不要求线程安全；hashmap，hashtable区别

77. hashset的实现原理 因为别人知道源码怎么实现的，故意构造相同的hash的字符串进行攻击，怎么处理 如果要去除后还要有序

78. 面向对象的三个特性

79. arraylist linkedlist vector的区别

80. 重写和重载

81. String有重写Object的hashcode和toString吗;String类添加功能，如何设计，可否继承

82. 对象深拷贝与浅拷贝

83. Java的反射原理

84. 队列 set map 区别

85. abstract 等关键字的作用，什么时候用

86. 子类中如何调用父类的构造器，如果不用super关键字呢？有其他的方式吗？

87. java中所有类的父类是什么？他都有什么方法

88. 解析XML的几种方式的原理与特点：DOM、SAX、PULL；Dom4J以及SAX的区别，什么时候用，怎么用

89. for each和正常for的用在不同数据结构（ArrayList、set、hashmap）上的效率区别

90. static class和non-static class的区别

91. String s=new String("abc")分别在堆栈上新建了哪些对象

92. Java 集合的快速失败机制 “fail-fast”

93. 手写一个ArrayList类，实现add，remove，等基本的方法；对这个ArrayList进行改进，使之可以应对并发的场景

94. Collections.sort函数jdk7 和 jdk8 分别怎么实现的

95. 为什么匿名内部类的变量必须用final修饰，编译器为什么要这么做，否则会出现什么问题；如何访问在其外面定义的变量？

96. PriorityQueue实现原理 替代：用TreeMap复杂度太高，有没有更好的方法。hash方法，但是队列不是定长的，如果改变了大小要rehash代价太大，还有什么方法？用堆实现，那每次get put复杂度是多少（lgN）

97. SimpleDateFormat 线程安全

98. Java代码编译过程

99. comparable接口和comparator接口实现比较的区别和用法

100. Arrays.sort

101. protected权限能否被包外访问；几种访问权限

102. linkedhashmap和hashmap，treemap的区别

100. BIO、NIO、AIO 重点NIO实现原理 NIO并不是严格意义上的非阻塞IO而应该属于多路复用IO；nio堆外内存，AIO在工程中如何实现的？（大概说了下ajax的回调函数），又问回调函数具体是怎么实现的（传递函数指针），NIO DirectMemory是否了解

101. synchronized原理，synchronized修饰静态变量和普通变量的区别，修饰普通方法和类方法的区别。和Lock对比着说，说到各自的优缺点，synchronized从最初性能差到jdk高版本后的锁膨胀机制，大大提高性能，再说底层实现，Lock的乐观锁机制，通过AQS队列同步器，调用了unsafe的CAS操作，CAS函数的参数及意义；同时可以说说synchronized底层原理，jvm层的moniter监视器，对于方法级和代码块级，互斥原理的不同，+1-1可重入的原理等lock的原理和实现，lock和synchronize的区别

102. concurrenthashmap（1.7 1.8区别）

103. 线程池原理，线程运行完后会消失吗？线程运行完后处于什么状态？怎么知道线程处于什么状态？线程池的构造类的方法的5个参数的具体意义？IO密集和CPU密集两种情况下，线程池里的线程数应该怎么设置，自己设计一个线程池

104. Condition，await signal

105. Java同步机制有哪几种

106. Java内存模型JMM

107. volatile：happens-before 内存屏障 指令重排 内存可见性

108. 线程的状态

109. 生产者消费者模式的几种实现

110. ThreadLocal

111. JVM锁的几种状态（轻量级锁、重量级锁）

112. 自旋锁 自旋锁会死锁吗

113. JUC的几种并发工具，用法和原理

114. CAS，原理 底层是JNI；ABA问题；JNI下cpu和寄存器层面是怎么实现的

115. 原子操作 AtomicInteger的实现原理，主要能说清楚CAS机制并且AtomicInteger是如何利用CAS机制实现的

116. 排他锁与共享锁

117. 读写锁

118. ReentrantLock底层的实现原理；lock和trylock的区别

119. AQS

120. 公平锁非公平锁介绍

121. 线程的几种实现方式

122. 三个线程轮流打印ABC十次

123. wait()和sleep()的区别

124. 如果两个线程都使用一个ByteBuf 怎么保证它的安全

125. ForkJoinPool，ForkJoin框架

126. 判断线程是否销毁

127. yield功能

128. 给定三个线程t1,t2,t3，如何保证他们依次执行

129. ArrayBlockingQueue和LinkedBlockingQueue 原理

130. JVM线程死锁，你该如何判断是因为什么？如果用VisualVM，dump线程信息出来，会有哪些信息？

131. 类加载机制（双亲委派、加载过程） 反双亲设计，类隔离

132. JVM 分区、GC算法：分代，分代又包含标记清除、标记整理、复制、GC对象存活、GC垃圾收集器（尤其是CMS、G1）年轻代为什么分为8比1

133. Serial 与 Parallel GC之间的不同之处

134. OOM；方法区OOM时的异常；查看dump文件，怎么查看，具体命令记得吗，答jstack 具体怎么用的

135. 四种引用 

136. JVM的内存参数；xmx,xms,xmn,xss参数你有调优过吗，设置大小和原则你能介绍一下吗？；Xss默认大小，在实际项目中你一般会设置多大

137. JVM 逃逸分析

138. JVM中对象组成

139. JVM指令（尤其是与并发有关的）

140. 竟态条件

141. 内存泄漏发生在哪 用过哪些调试java内存工具；非静态内部类会导致内存泄露，说了内存泄露的解决

142. 方法栈

143. 热加载的原理 osgi实现,也就是自己实现classloader,还是来自周志明的那本jvm

144. Minor GC与Full GC分别在什么时候发生；如何手动触发全量回收垃圾，如何立即触发垃圾回收；minor gc回收新生代，major gc回收老年代，fullgc是全堆回收，老年代+新生代。

145. 如何把java内存的数据全部dump出来

146. jstack、jmap、jconsole等工具 可视化工具使用；如何线上排查JVM的相关问题？

147. 如果一个接口调用很慢，原因是，如何定位，没有日志的话：假设一下，复现问题，dump查看内存，查看监控日志

148. StackOverFlow 栈溢出如何解

149. JKD1.8的JVM指令集上有什么更新吗对比1.7 

150. JVM如何确立每个类在JVM的唯一性；类的全限定名和加载这个类的类加载器的ID

151. 字节码的编译过程

152. 查看jvm虚拟机里面堆、线程的信息，你用过什么命令？

153. 在生产线Dump堆分析程序是否有内存及性能问题
