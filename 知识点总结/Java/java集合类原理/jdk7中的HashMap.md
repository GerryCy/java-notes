





[HashMap中的存储结构图示](#HashMap中的存储结构图示)

[HashMap中的成员变量以及含义](#HashMap中的成员变量以及含义)

[HashMap构造方法](#HashMap构造方法)

[怎样确定元素在桶中的位置](#怎样确定元素在桶中的位置)

[put方法分析](#put方法分析)

[resize方法分析](#resize方法分析)

[get方法分析](#get方法分析)

## HashMap中的存储结构图示

​	jdk1.7的HashMap采用数组+单链表实现，尽管定义了hash函数来避免冲突，但因为数组长度有限，还是会出现两个不同的Key经过计算后在数组中的位置一样，1.7版本中采用了链表来解决。

![](../../../assert/7hash.png)

​	从上面的简易示图中也能发现，如果位于链表中的结点过多，那么很显然通过key值依次查找效率太低，所以在1.8中对其进行了改良，采用数组+链表+红黑树来实现，当链表长度超过阈值8时，将链表转换为红黑树.具体细节参考我上一篇总结的 [深入理解jdk8中的HashMap](https://juejin.im/post/5d37b5475188251b4b32b993 )

​	从上面图中也知道实际上每个元素都是Entry类型，所以下面再来看看Entry中有哪些属性(在1.8中Entry改名为Node，同样实现了Map.Entry)。 

```java
//hash标中的结点Node,实现了Map.Entry
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;
	//Entry构造器，需要key的hash，key，value和next指向的结点
    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }

    public final K getKey() { return key; }

    public final V getValue() { return value; }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    //equals方法
    public final boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;
        Object k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            Object v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }
	//重写Object的hashCode
    public final int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    public final String toString() {
        return getKey() + "=" + getValue();
    }

	//调用put（k，v）方法时候，如果key相同即Entry数组中的值会被覆盖，就会调用此方法。
    void recordAccess(HashMap<K,V> m) {
    }

    //只要从表中删除entry，就会调用此方法
    void recordRemoval(HashMap<K,V> m) {
    }
}
```



## HashMap中的成员变量以及含义

```java
//默认初始化容量初始化=16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

//最大容量 = 1 << 30
static final int MAXIMUM_CAPACITY = 1 << 30;

//默认加载因子.一般HashMap的扩容的临界点是当前HashMap的大小 > DEFAULT_LOAD_FACTOR * 
//DEFAULT_INITIAL_CAPACITY = 0.75F * 16
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//默认是空的table数组
static final Entry<?,?>[] EMPTY_TABLE = {};

//table[]默认也是上面给的EMPTY_TABLE空数组，所以在使用put的时候必须resize长度为2的幂次方值
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

//map中的实际元素个数 != table.length
transient int size;

//扩容阈值，当size大于等于其值，会执行resize操作
//一般情况下threshold=capacity*loadFactor
int threshold;

//hashTable的加载因子
final float loadFactor;

/**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     */
transient int modCount;

//hashSeed用于计算key的hash值，它与key的hashCode进行按位异或运算
//hashSeed是一个与实例相关的随机值，用于解决hash冲突
//如果为0则禁用备用哈希算法
transient int hashSeed = 0;
```



## HashMap构造方法

​	我们看看HashMap源码中为我们提供的四个构造方法。

```java
//(1)无参构造器：
//构造一个空的table，其中初始化容量为DEFAULT_INITIAL_CAPACITY=16。加载因子为DEFAULT_LOAD_FACTOR=0.75F
public HashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}
```

```java
//(2)指定初始化容量的构造器
//构造一个空的table，其中初始化容量为传入的参数initialCapacity。加载因子为DEFAULT_LOAD_FACTOR=0.75F
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

```java
//(3)指定初始化容量和加载因子的构造器
//构造一个空的table，初始化容量为传入参数initialCapacity，加载因子为loadFactor
public HashMap(int initialCapacity, float loadFactor) {
    //对传入初始化参数进行合法性检验，<0就抛出异常
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    //如果initialCapacity大于最大容量，那么容量=MAXIMUM_CAPACITY
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //对传入加载因子参数进行合法检验，
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        //<0或者不是Float类型的数值，抛出异常
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
	//两个参数检验完了，就给本map实例的属性赋值
    this.loadFactor = loadFactor;
    threshold = initialCapacity;
    //init是一个空的方法，模板方法，如果有子类需要扩展可以自行实现
    init();
}
```

​	从上面的这3个构造方法中我们可以发现虽然指定了初始化容量大小，但此时的table还是空，是一个空数组，且扩容阈值threshold为给定的容量或者默认容量(前两个构造方法实际上都是通过调用第三个来完成的)。在其put操作前，会创建数组（跟jdk8中使用无参构造时候类似）.

```java
//(4)参数为一个map映射集合
//构造一个新的map映射，使用默认加载因子，容量为参数map大小除以默认负载因子+1与默认容量的最大值
public HashMap(Map<? extends K, ? extends V> m) {
    //容量：map.size()/0.75+1 和 16两者中更大的一个
    this(Math.max(
        	(int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                  DEFAULT_INITIAL_CAPACITY), 
         DEFAULT_LOAD_FACTOR);
    inflateTable(threshold);
    //把传入的map里的所有元素放入当前已构造的HashMap中
    putAllForCreate(m);
}
```

​	这个构造方法便是在put操作前调用inflateTable方法，这个方法具体的作用就是创建一个新的table用以后面使用putAllForCreate装入传入的map中的元素，这个方法我们来看下，注意刚也提到了此时的threshold扩容阈值是初始容量。下面对其中的一些方法进行说明

### (1)inflateTable方法说明

```java
private void inflateTable(int toSize) {
    //返回不小于number的最小的2的幂数，最大为MAXIMUM_CAPACITY,类比jdk8的实现中的tabSizeFor的作用
    int capacity = roundUpToPowerOf2(toSize);
	//扩容阈值为：(容量*加载因子)和(最大容量+1)中较小的一个
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    //创建table数组
    table = new Entry[capacity];
    initHashSeedAsNeeded(capacity);
}
```

### (2)roundUpToPowerOf方法说明

```java
private static int roundUpToPowerOf2(int number) {
    //number >= 0，不能为负数，
    //(1)number >= 最大容量：就返回最大容量
    //(2)0 =< number <= 1：返回1
    //(3)1 < number < 最大容量：
    return number >= MAXIMUM_CAPACITY
        ? MAXIMUM_CAPACITY
        : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
}
//该方法和jdk8中的tabSizeFor实现基本差不多
public static int highestOneBit(int i) {
    //因为传入的i>0,所以i的高位还是0，这样使用>>运算符就相当于>>>了，高位0。
    //还是举个例子，假设i=5=0101
    i |= (i >>  1); //（1）i>>1=0010；（2）i= 0101 | 0010 = 0111
    i |= (i >>  2); //（1）i>>2=0011；（2）i= 0111 | 0011 = 0111
    i |= (i >>  4); //（1）i>>4=0000；（2）i= 0111 | 0000 = 0111
    i |= (i >>  8); //（1）i>>8=0000；（2）i= 0111 | 0000 = 0111
    i |= (i >> 16); //（1）i>>16=0000；（2）i= 0111 | 0000 = 0111
    return i - (i >>> 1); //（1）0111>>>1=0011（2）0111-0011=0100=4
    //所以这里返回4。
    //而在上面的roundUpToPowerOf2方法中，最后会将highestOneBit的返回值进行 << 1 操作，即最后的结果为4<<1=8.就是返回大于number的最小2次幂
}
```

### (3)putAllForCreate方法说明

​	该方法就是遍历传入的map集合中的元素，然后加入本map实例中。至于putForCreate方法后面在说put操作的时候会说到。

```java
private void putAllForCreate(Map<? extends K, ? extends V> m) {
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        putForCreate(e.getKey(), e.getValue());
}
```



## 怎样确定元素在桶中的位置

​	1.7中的hash算法和1.8的实现是不一样的，而hash值又关系到我们put新元素的位置、get查找元素、remove删除元素的时候去通过indexFor查找下标。所以我们来看看这两个方法

### 	(1)hash方法

```java
final int hash(Object k) {
    int h = hashSeed;
    //默认是0，不是0那么需要key是String类型才使用stringHash32这种hash方法
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }
    h ^= k.hashCode();
    //将高位的值移位到低位参与运算，是的高位值变化能够影响到hash结果
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

​	根据hash值确定在table中的位置，length为2的倍数 HashMap的扩容是基于2的倍数来扩容的，从这里可以看出，对于indexFor方法而言，其具体实现就是通过一个计算出来的code值和数组长度-1做位运算，那么对于2^N来说，长度减一转换成二进制之后就是全一（长度16，len-1=15,二进制就是1111），所以这种设定的好处就是说，对于计算出来的code值得每一位都会影响到我们索引位置的确定，其目的就是为了能让数据更好的散列到不同的桶中

### 	(2)indexFor方法

```java
static int indexFor(int h, int length) {
    //还是使用hash & (n - 1)计算得到下标
    return h & (length-1);
}
```





## put方法分析

```java
public V put(K key, V value) {
    //我们知道Hash Map有四中构造器
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}


private V putForNullKey(V value) {
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}

private void putForCreate(K key, V value) {
    int hash = null == key ? 0 : hash(key);
    int i = indexFor(hash, table.length);

    /**
         * Look for preexisting entry for key.  This will never happen for
         * clone or deserialize.  It will only happen for construction if the
         * input Map is a sorted map whose ordering is inconsistent w/ equals.
         */
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            e.value = value;
            return;
        }
    }

    createEntry(hash, key, value, i);
}

private void putAllForCreate(Map<? extends K, ? extends V> m) {
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        putForCreate(e.getKey(), e.getValue());
}
```







## resize方法分析

```java
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
```







## get方法分析

```java
    public V get(Object key) {
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }

    /**
     * Offloaded version of get() to look up null keys.  Null keys map
     * to index 0.  This null case is split out into separate methods
     * for the sake of performance in the two most commonly used
     * operations (get and put), but incorporated with conditionals in
     * others.
     */
    private V getForNullKey() {
        if (size == 0) {
            return null;
        }
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null)
                return e.value;
        }
        return null;
    }
```





















