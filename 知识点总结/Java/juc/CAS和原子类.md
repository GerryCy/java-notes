## CAS思想

### CAS概述

​	CAS需要3个操作数，分别是内存位置V、旧的预期值A和新值B，在CAS执行的时候，当且仅当内存位置V上的值符合旧的值A的时候，采用新值B更新V处的值，否则不做更新（无论是否更新了V，都会返回V的旧值）。比如在JUC下的atomic包中就大量使用这种算法。

### CAS在atomic包中的使用

我们简单看一下AtomicInteger包的部分源码，来了解一下CAS在atomic包下的应用

```java
/*
 * AtomicInteger下面的部分源码
 */
private static final Unsafe unsafe = Unsafe.getUnsafe(); //获取unsafe实例
private static final long valueOffset; //value 的内存偏移量
private volatile int value; //存取的value值
//获取AtomicInteger 的value属性的内存偏移量
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
//compareAndSet就是使用CAS思想更新变量的值，调用unsafe类下的本地方法
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
/*
 * Unsafe类下的一个本地方法
 */
//方法中有四个操作数，其中obj表示对象内存的位置，valueOffset表示对象中存储变量的偏移量，expect表示变量的预期
//值，update表示更新值。操作含义就是，若果对象obj中内存偏移量为valueOffset的变量值为expect,则使用新值update
//值替更新的值expect，这是处理器提供的一个原子指令。
public final native boolean compareAndSwapInt(Object obj, long valueOffset, int expect, int update);
```

### CAS的问题

​	ABA问题：假如线程1 使用CAS修改初始值为A的变量X，那么线程1会首先回去当前变量X的值(A)，然后使用CAS操作尝试修改X的值为B，如果使用CAS修改成功了，那么程序一定执行正确了吗？在往下的假设看，如果线程I在获取变量X的值A后，在执行CAS之前线程II使用CAS修改变量X的值为B然后由修改回了A。这时候虽然线程I执行CAS时候X的值依旧是A但是这个A已经不是线程I获取时候的A了，这就是ABA问题。

​	ABA产生的原因是变量的状态值产生了环形转换，即变量值从A->B，然后又从B->A。jdk中提供了带有标记的原子类AtomicStampedReference(时间戳原子引用)通过控制变量的版本保证CAS的正确性。

## 使用CAS自定义一个显示锁

## 原子变量

​	JUC包中提供了很多原子操作类，这些类都是通过上面说到的CAS算法来实现的，相比较使用锁来实现原子性操作CAS在性能上有很大提高。由于原子操作类的原理都大致相同，所以下面分析简单看一下每一种原子类的实现原理来进一步了解原子操作类。参考总结篇之[从同步原语看非阻塞同步以及Java中的应用](https://www.cnblogs.com/fsmly/p/11019223.html)，其中讲到了同步原语以及Java中的Unsafe、CAS以及原子变量原理实现。

### AtomicInteger

参考上面的[CAS在atomic包中的使用](#CAS在atomic包中的使用)使用AtomicInteger举的例子。

### AtomicBoolean

AtomicBoolean实际上使用的是int类型的变量表示true(1)和false(0)，我们直接看一下AtomicBoolean的部分源码

```java
//获取unsafe实例
private static final Unsafe unsafe = Unsafe.getUnsafe(); 
//属性value的内存偏移量
private static final long valueOffset;
//初始化的时候，计算value的内存偏移量，并赋给valueOffset
static {
try {
    valueOffset = unsafe.objectFieldOffset
   		 (AtomicBoolean.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
//实际存的值(0?1)
private volatile int value;
//带有参数的构造器，可以发现initialValue为true，那么value=1；initialValue=false，value=0
public AtomicBoolean(boolean initialValue) {
        value = initialValue ? 1 : 0;
}
//默认无参构造器，显然value默认值为0，即false
public AtomicBoolean() {
}
//返回value的值(0==false,1==true)
public final boolean get() {
        return value != 0;
}
//使用Unsafe类的compareAndSwapInt完成原子交换，关于compareAndSwapInt的介绍在上面AtomicInteger中说到过，
//这里不重复说明
public final boolean compareAndSet(boolean expect, boolean update) {
    int e = expect ? 1 : 0;
    int u = update ? 1 : 0;
    return unsafe.compareAndSwapInt(this, valueOffset, e, u);
}
//(1)prev = get();先获取旧值value，
//(2)调用上面compareAndSet原子的更新value值
//(3)如果更新成功compareAndSet会返回true，然后退出循环，否则一直自旋尝试更新知道成功
//(4)返回prev(旧值)
public final boolean getAndSet(boolean newValue) {
    boolean prev;
    do {
        prev = get();
    } while (!compareAndSet(prev, newValue));
    return prev;
}
```

### AtomicLong

```java
//获取unsafe实例
private static final Unsafe unsafe = Unsafe.getUnsafe();
//存储的类属性value的内存偏移量
private static final long valueOffset;
//主要差别在这里：VM_SUPPORTS_LONG_CAS被JVM调用，会查看是不是JVM支持对long类型的compareAndSwap，虽然
//Unsafe.compareAndSwapLong方法适用于任何一种情况，但是应该在Java级别处理一些构造以避免使用用户级别锁来进行对long类型的锁定。

//long:64位，在数据总线传递的时候可能不是原子性的(32位CPU或者其他)，分成高位、低位去传递。
static final boolean VM_SUPPORTS_LONG_CAS = VMSupportsCS8();
private static native boolean VMSupportsCS8();
//初始化的时候，计算value的内存偏移量，并赋给valueOffset
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicLong.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
public final boolean compareAndSet(long expect, long update) {
    return unsafe.compareAndSwapLong(this, valueOffset, expect, update);
}
```

### AtomicReference

​	主要就是解决某些场合基本原子类型不能够解决的问题(比如想某个封装类也具备原子性的更新......)，AtomicReference就是解决这个问题。我们看一下AtomicReference的部分代码。和上面的基本原子类型差不多(就是AtomicReference<V>可以解决对封装类的原子性操作了）。

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicReference.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

private volatile V value;
public final boolean compareAndSet(V expect, V update) {
    return unsafe.compareAndSwapObject(this, valueOffset, expect, update);
}
```

### AtomicStampReference

​	主要是为了解决CAS带来的另一个问题[CAS的问题](#CAS的问题)，AtomicStampReference通过时间戳或者版本号的机制来解决CAS的ABA问题，我们先看看AtomicStampReference的部分源码实现

```java
//通过静态内部类的方式存储要设置为原子类型的引用类型
private static class Pair<T> {
    final T reference; //泛型T：代表构造的时候的传递的类型
    final int stamp; //版本号
    private Pair(T reference, int stamp) { //私有构造，不对外暴露，只在AtomicStampedReference内部使用of方法创建Pair对象
        this.reference = reference;
        this.stamp = stamp;
    }
    static <T> Pair<T> of(T reference, int stamp) {
        return new Pair<T>(reference, stamp); //每次调用of方法都会返回一个新的reference+stamp
    }
}
//volatile修饰的Pair类型
private volatile Pair<V> pair;
//获取unsafe示例
private static final sun.misc.Unsafe UNSAFE = sun.misc.Unsafe.getUnsafe();
//计算AtomicStampedReference的属性pair的内存偏移量
private static final long pairOffset =
    objectFieldOffset(UNSAFE, "pair", AtomicStampedReference.class);
//casPair：使用Unsafe类的compareAndSwapObject原子的更新pair
private boolean casPair(Pair<V> cmp, Pair<V> val) {
    return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val);
}
//构造方法，传入初始对象引用和版本号stamp
public AtomicStampedReference(V initialRef, int initialStamp) {
    pair = Pair.of(initialRef, initialStamp); //使用of方法构造一个pair对象
}
//返回构造的对象引用
public V getReference() {
    return pair.reference;
}
//返回版本号stamp
public int getStamp() {
    return pair.stamp;
}
//返回包装的对象引用，向传入的数组参数中设置当前的版本号stamp
public V get(int[] stampHolder) {
    Pair<V> pair = this.pair;
    stampHolder[0] = pair.stamp;
    return pair.reference;
}
//
public boolean compareAndSet(V   expectedReference,
                             V   newReference,
                             int expectedStamp,
                             int newStamp) {
    Pair<V> current = pair;
    //(1)先比较传入expectedReference是不是当前正确的reference(这个reference构造的时候传入的类类型引用)
    //(2)比较传入的expectedStamp是不是当前正确的stamp
    //(3)如果(1)和(2)都为true
    //	a.会比较更新的newReference和当前的reference是不是相同
    //	b.比较更新的newStamp和当前的stamp是不是相同
    //	c.如果上面的a和b都为true，那么就不做更新
    //	d.如果上面的a和b有一个为false，就会使用casPair原子性的更新reference和stamp
    return
        expectedReference == current.reference 
        && expectedStamp == current.stamp
        && (
           (newReference == current.reference 
            && newStamp == current.stamp)
            || casPair(current, Pair.of(newReference, newStamp)
           )
    );
}

//无条件的将当前被包装的对象设置为传入的值(reference和stamp)
public void set(V newReference, int newStamp) {
    Pair<V> current = pair;
    if (newReference != current.reference || newStamp != current.stamp)
        this.pair = Pair.of(newReference, newStamp); //并且当前pair由新的值替换
}
```

### AtomicIntegerArray

​	主要是提供原子的方式操作数组里的整形，AtomicIntegerArray的构造方法和常用的方法。

```java
public AtomicIntegerArray(int length) {
    array = new int[length];
}
/**
 * 如果参数为一个array，那么AtomicIntegerArray会将当前数组复制一份，所以对AtomicIntegerArray内部数组元素
 * 操作的时候，不会影响到原来的array的值
 * @throws NullPointerException if array is null
 */
public AtomicIntegerArray(int[] array) {
    this.array = array.clone();
}
public final int addAndGet(int i, int delta) {
    //以原子的方式将输入值与数组中索引为i的元素相加，返回更新后的结果
}
public final boolean compareAndSet(int i, int expect, int update) {
    //如果当前值等于预期值，则以原子方式将数组位置为i的元素设置为update值，并返回true；否则返回false
}
```

### AtomicLongArray

​	原子更新长整型数组里面的元素。和AtomicIntegerArray不同的地方就是方法使用C-A-S-Long的形式。

### AtomicReferenceArray

​	原子更新长整型数组里面的元素。和AtomicIntegerArray不同的地方就是方法使用C-A-S-Obj的形式。	

### AtomicIntegerFieldUpdater

​	原子更新整形字段(int，如果使用Integer会抛异常)的。比如下面的例子（将SimpleObject类中的整形属性id添加到了原子更新器上）.

```java
//具体的使用分为两步：(1)使用AtomicIntegerFieldUpdater的静态方法newUpdater创建一个更新器，同时在其中设置
//是哪个类的哪个属性。(2)所添加的那个类的属性需要使用public volatile修饰
public class TestAtomicIntegerFieldUpdater {
    private static AtomicIntegerFieldUpdater<SimpleObject> updater = 
        AtomicIntegerFieldUpdater.newUpdater(SimpleObject.class, "id");

    public static void main(String[] args) {
        SimpleObject simpleObject = new SimpleObject("test1",1);
        for (int i = 0; i < 3; i++) {
            new Thread(){
                @Override
                public void run() {
                    for (int j = 0; j < 30; j++) {
                        System.out.println(Thread.currentThread().getName() + 
                                           " ==>id: "+updater.getAndIncrement(simpleObject));
                    }
                }
            }.start();
        }
    }

    private static class SimpleObject {
        private String name;
        public volatile int id;

        public SimpleObject(String name, int id) {
            this.name = name;
            this.id = id;
        }
    }
}
```

### AtomicLongFieldUpdater

​	原子更新长整型字段(long)的更新器。

### AtomicReferenceFieldUpdate

​	原子更新引用类型的更新器。

```java
public class TestFieldUpdater {

    //java.lang.IllegalAccessException: 当所使用的类的属性为private(如果是当前类，也可以是private的)
    @Test
    public void testIllegalAccessException() {
        AtomicIntegerFieldUpdater<SimpleObject> updater = AtomicIntegerFieldUpdater.newUpdater(SimpleObject.class, "id");
        SimpleObject simpleObject = new SimpleObject("test1", 1);
        updater.compareAndSet(simpleObject, 1, 2);
    }

    //java.lang.ClassCastException: 使用更新器的方法时候，如果传递的为null
    @Test
    public void testClassCastException() {
        AtomicIntegerFieldUpdater<SimpleObject> updater = AtomicIntegerFieldUpdater.newUpdater(SimpleObject.class, "id");
        SimpleObject simpleObject = new SimpleObject("test1", 1);
        updater.compareAndSet(null, 1, 2);
    }

    //java.lang.NoSuchFieldException: 使用的类的字段名传入错误
    @Test
    public void testNoSuchFieldException() {
        AtomicIntegerFieldUpdater<SimpleObject> updater = AtomicIntegerFieldUpdater.newUpdater(SimpleObject.class, "id1");
        SimpleObject simpleObject = new SimpleObject("test1", 1);
        updater.compareAndSet(null, 1, 2);
    }

    //java.lang.ClassCastException: 当传入的类型不匹配的时候(Integer->Long)
    @Test
    public void testFieldTypeInvalid() {
        AtomicReferenceFieldUpdater<SimpleObject, Long> updater = AtomicReferenceFieldUpdater.newUpdater(SimpleObject.class, Long.class, "id");
        SimpleObject simpleObject = new SimpleObject("test1", 1);
        updater.compareAndSet(simpleObject, 1L, 2L);
    }

    //java.lang.IllegalArgumentException: Must be volatile type: 必须被volatile修饰
    @Test
    public void testVolatileType() {
        AtomicReferenceFieldUpdater<SimpleObject, Integer> updater = AtomicReferenceFieldUpdater.newUpdater(SimpleObject.class, Integer.class, "id");
        SimpleObject simpleObject = new SimpleObject("test1", 1);
        updater.compareAndSet(simpleObject, 1, 2);
    }
}
```

### 原子更新字段类使用场景

(1)需要让类的属性操作具备原子性

(2)不想使用锁(lock或者synchronized)

(3)大量需要原子类型修饰的对象(相比较会比直接使用AtomicStampedReference节省内存)

## Unsafe类

