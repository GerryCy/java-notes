​	上一篇总结了[ConcurrentLinkedQueue源码分析](https://juejin.im/post/5d4789ca51882519ac307a6f)这个并发队列，其实现主要依赖于CAS算法，在并发情况下是非阻塞的，下面我们看看jdk为我们提供的使用基于独占锁和单链表实现的阻塞队列LinkedBlockingQueue。先简单总结一下LinkedBlockingQueue的特点：

- 使用单向链表实现，链表结点类型为Node；
- 使用head和tail维护链表的头节点（哨兵）和尾节点；
- 一个初始化值为0的原子变量count，用来记录队列的元素个数；
- 使用`takeLock`、`putLock`两个ReentrantLock实例，分别控制元素入队和出队的原子性，具体表现为
  - takeLock控制同时只有一个线程能够从队列头部获取元素，其他线程阻塞等待
  - putLock控制同时只有一个线程可以获取锁，在队列尾部添加元素，其他线程等待

- notEmpty、notFull是条件变量(`Condition`)，内部使用条件队列用来存放进队和出队的时候被阻塞的线程

下面我们从`LinkedBlockingQueue`的源码上来简单分析一下这个并发队列的实现细节。

## LinkedBlockingQueue的属性

```java
//单链表中的结点类型Node
static class Node<E> {
    //单链表中的item指针
    E item;
    //单链表中的next结点
    Node<E> next;
    //Node结点的构造方法
    Node(E x) { item = x; }
}
//队列的容量，当调用的是无参构造方法的时候，其值是Integer.MAX_VALUE
private final int capacity;
//使用原子类型，用以记录当前队列中的结点个数
private final AtomicInteger count = new AtomicInteger();
//链表的头节点:head.item=null
transient Node<E> head;
//链表的尾节点
private transient Node<E> last;
//take, poll使用的lock
private final ReentrantLock takeLock = new ReentrantLock();
//take方法的等待队列
private final Condition notEmpty = takeLock.newCondition();
//put、offer使用的lock
private final ReentrantLock putLock = new ReentrantLock();
//put方法的等待队列
private final Condition notFull = putLock.newCondition();
```

## LinkedBlockingQueue的构造方法

（1）无参构造

```java
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE); //无参构造方法，队列的容量为Integer.MAX_VALUE
}
```

（2）指定容量的构造器

```java
public LinkedBlockingQueue(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity; //指定容量
    last = head = new Node<E>(null); //last和head在初始时候指向的都是item为null的结点
}
```

（3）参数为Collection的集合

```java
public LinkedBlockingQueue(Collection<? extends E> c) {
    this(Integer.MAX_VALUE); //创建的队列初始容量为Integer.MAX_VALUE
    final ReentrantLock putLock = this.putLock; //获取putLock
    putLock.lock(); // Never contended, but necessary for visibility
    try {
        int n = 0;
        for (E e : c) { //foreach遍历，将集合中的数据加入队列中
            if (e == null)
                throw new NullPointerException();
            if (n == capacity)
                throw new IllegalStateException("Queue full");
            enqueue(new Node<E>(e));
            ++n;
        }
        count.set(n); //更新原子变量count的值
    } finally {
        putLock.unlock();
    }
}
```

## 入队offer方法

```java
public boolean offer(E e) {
    //传入元素为null，抛出异常，所以LinkedBlockingQueue不能含有
    if (e == null) throw new NullPointerException();
    //获得当前的count数值
    final AtomicInteger count = this.count;
    //获取当前的count值之后，和队列的容量进行比较，如果等于当前队列容量那么直接返回false
    if (count.get() == capacity)
        return false;
    //size大小
    int c = -1;
    //创建一个新的结点，结点中的item为传入的元素e
    Node<E> node = new Node<E>(e);
    //获取putLock，这个锁控制同时只有一个线程可以获取锁，在队列尾部添加元素，其他线程等待
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        //count值小于容量
        if (count.get() < capacity) {
            //入队列
            enqueue(node);
            //更新count的值
            c = count.getAndIncrement();
            //更新后的count小于容量，那么唤醒别的阻塞在putLock上的线程
            if (c + 1 < capacity)
                notFull.signal();
        }
    } finally {
        putLock.unlock();
    }
    //当前的count为0，说明在putLock的时候，队列中只有一个元素，所以需要调用signalNotEmpty方法
    if (c == 0)
        signalNotEmpty();
    return c >= 0;
}
private void enqueue(Node<E> node) {
    // assert putLock.isHeldByCurrentThread();
    // assert last.next == null;
    last = last.next = node;
}
private void signalNotEmpty() {
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
}
```

