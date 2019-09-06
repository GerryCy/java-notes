## Java相关

1. 线程安全、valotile、偏向锁，自旋锁什么的，问了个ABA问题；
2. jvm 里 new 对象时，堆会不会发生抢占？那你怎么设计jvm的堆的线程安全？
3. java中是怎样自动装箱拆箱的
4. Spring中的IOC怎样解决循环依赖问题
5. Java调度进程和线程
6. String，StringBuffer，StringBuilder区别以及使用场景
7. 线程池、线程池的拒绝策略？线程池工作原理
8. java的锁；
9. 死锁，如何预防死锁，如果已经死锁了怎么解决；
10. 不使用锁怎么实现线程安全（copyAndWrite)；
11. beanFactory 和 FactoryBean 有啥区别；
12. spring IOC，解释一下，举个例子，如果不使用IOC用你的方式实现怎么实现；spring aop；
13. hashmap底层实现，自定义一个对象，插入过程（什么使用hasdcode，什么时候使用equals.....)；hashmap扩容多大、hash碰撞解决办法
14. concurrentHashMap说一下喽，1.8之后改进为啥用红黑树，不使用其他树。。。。。（数据过多，链表扩展成数组，即添加红黑树。查找方便，主要是红黑树维持着一种相对平衡没有AVL那么频繁，代价不会那么大）  
15. 不用线程的集合工具类的锁，如何保证hashMap的安全。。（想问自己加锁机制，你如何实现）
16. equals和==区别，为啥重写equals一定要重写hashcode
17. 字符串拼接操作、不用String的加号
18. SpringMVC中容器的bean能访问到Spring容器中的bean吗？反过来呢；事务传播
19. 线程wait了，被notifyall了，变成啥状态了，线程wait了会不会释放锁
20. mybatis的#好处
21. threadlocal，说说（结合源码）
22. 多态(为什么用多态)、多态的底层的原理
23. spring框架、依赖注入、spring注解
24. synchronized锁住对象和.class区别，synchronized原理、对象头除了存线程ID，还能存什么内容
25. jvm垃圾回收 很详细  内存模型
26. redis底层数据结构
27. Servlet相关接口、Servlet里面有哪些方法、doGet, doPost区别
28. mybits一二级缓存
29. NIO BIO AIO的区别
30. redis持久化区别
31. 类加载机制双亲委派和打破双亲委派、对象生命周期、自己实现类加载器
32. Java集合了解哪些？说了list set map的特点，每个中常用的类型，ArrayList与linklist的区别，查询与插入的时间复杂度，
33. 线程的创建方式，callable与runnable的区别
34. Java内存模型、JVM内存机制，垃圾回收机制、 gc 垃圾收集器知道哪些、java虚拟机回收算法。回收机制。回收器，年轻代到达老年代（讲了很细）。回收过程，jvm中什么样的可以被回收。对象从进入堆区到死亡的全流程
35. 工厂模式
36. 新生代老年代为什么这么划分
37. 单例模式 代理模式 springAOP、模版模式，动态代理模式、静态代理、观察者模式
38. CountDownLatch、CyclicBarrier区别、场景题目
39. Docker讲一下，和虚拟机比为啥好，为啥非用docker
40. java实现多线程消费者和生产者
41. 虚拟机运行时数据区。其中有什么数据。哪些区域是共享的。是私有的。
42. AQS、Lock原理、CLH同步队列怎么实现非公平与公平的？
43. 一个请求在 SpringMVC 中的流程
44. Java反射，泛型的原理，怎么实现的
45. Java垃圾回收算法有几种，分别是什么?G1垃圾收集器过程及特点? CMS垃圾收集器采用哪种垃圾回收算法?CMS垃圾收集器垃圾回收具体过程?CMS垃圾收集器有几次Stop the Word，具体是什么情况发生?
46. CAS是什么，优缺点？
47. spring中autowired resource区别
48. Redis的基本数据结构，Set怎么实现的，跳表的插入删除查询时间复杂度
49. 一致性hash，作用是什么
50. 负载均衡算法有哪些

## 数据库

1. RDB,redis与mysql的区别

2. 深挖mysql，索引,慢查询优化知道的多一点，行锁表锁，Innodb和MyIsam的区别；

3. mysql的索引说一下，聚簇索引，联合索引；mysql索引、底层实现原理b树、b+树讲细一些、各种二叉树之类的为什么不能够很好地实现。而是使用b树或者b+树。索引优化。有哪些索引类型，建立索引的考量。

4. mysql隔离级别，读已提交和可重复读区别；

5. 如果缓存雪崩，大量请求直接打到数据库怎么办？；

6. 关系型数据库和非关系型数据库区别；

7. 数据库设计三大范式；

8. 缓存：怎么保证缓存与数据库缓存一致

9. 本地缓存如果用了 用了什么

10. 本地缓存如何与缓存一致

11. mysql索引，explain语句出来的结果中的type如果为index是啥意思，间隙锁锁的是数据还是索引，间隙锁解决的是啥问题。b 树，介绍B+树，B+树和B树的区别；

12. 幻读会在哪种隔离级别中出现，如何解决幻读

13. 项目里是否用到了事务？哪里用到了？单表更新是否需要事务？

14. 缓存雪崩

    ```
    a.缓存穿透的问题:
    解决方案:
    
    1.布隆过滤器 把所有可能存在的数据哈希到一个足够大的bitmap中,一个一定不存在的数据会被这个bitmap拦截掉
    
    2.如果一个查询返回的数据是空的,无论数据是否存在,还是系统故障,我们仍然将这个空结果进行缓存,但设置很短的过期时间
    
    b.缓存雪崩的问题: 设置缓存的时候采用了相同的过期时间,导致缓存在同一时刻失效,请求全部转发到db,db压力瞬间变大雪崩。
    解决方案
    
    1.用加锁或者队列的方式保证缓存的单线程写,避免失效时大量的并发请求落到底层的存储系统上
    
    2.将缓存的失效时间分散开,在原来失效时间的基础上加上一个随机值,比如说1-5分钟的随机,这样每一个缓存过期时间的重复率就会降低,很难引起集体失效的事件。
    
    c.缓存击穿的问题。对于一些设置了过期时间的key,这些key某些时间点被超高并发地访问,是非常热点的数据,缓存在某个时间点过期的时候,恰好这个时间节点对这个key有大量的并发请求过来,这些请求发现缓存的key过期就一般都会从后端数据库请求数据并且设置到缓存,大的并发请求就可能会压垮后端db
    
    解决方案
    
    1.使用互斥锁(mutex) , 在缓存失效的时候,判断拿出来的值为空,不立即去load db,而是先使用缓存工具的某些带有返回值的操作(比如说redis的setnx),去set一个mutex key,当操作返回成功的时候,在进行load db的操作并且设置回缓存,否则,就重试整个get缓存的方法。
    String get(String key){
        String value = reids.get(key);
        if(value == null){
            if(redis.setnx(key_mutex,"1")){
                reids.expire(key_mutex,3*60);
                value = db.get(key);
                redis.set(key,value);
                redis.delete(key_mutex);
            }else{
                //其他线程休息50ms后重试
                Thread.sleep(50);
                get(key);
            }
        }
    }
    2.设置热点数据永不过期
    
    a.从redis上来看,确实不设置过期时间,保证不会出现热点key过期的问题,也就是物理上的不过期
    
    b.从功能上看,我们把过期时间存在key对应的value里面,如果发现要过期了,通过一个后台的异步线程进行缓存的构建,设置"逻辑"过期时间
    String get(final String key){
        V v = redis.get(key);
        String value = v.getValue();
        long timeout = v.getTimeout();
        if(v.timeout <= System.currentTimeMillis()){
            //异步更新后台异常执行
            threadPool.execute(new Runnable(){
                public void run(){
                    String keyMutex = "mutex:"+key;
                    if(redis.setnx(keyMutex,"1")){
                        //设置三分钟的超时
                        redis.expire(keyMutex,3*60);
                        String dbValue = db.get(key);
                        redis.set(key,dbValue);
                        redis.delete(keyMutex);
                    }
                }   
            });
        }
    }
    
    
    3.提前使用互斥锁
    在value内部设置一个超时值(timeout1),timeout1比实际的memcache timeout(timeout2)小,当从cache读取到timeout1发现它已经过期的时候,马上延长timeout1并重新设置到cache。然后再从数据库加载数据并且设置到cache中
    v = memcache.get(key);
    if(v==null){
        if(memcache.add(key_mutex,3*60*1000) == true){
            value = db.get(key);
            memcache.set(key,value);
            memcache.delete(key_mutex);
        }else{
            sleep(50);
            retry();
        }else{
            if(v.timeout <= now()){
                if(memcache.add(key_mutex,3*60*1000) == true){
                    v.timeout += 3*60*1000;
                    memcache.set(key,v,KEY_TIMEOUT*2);
                     
                    //从数据库加载最新的数据
                    v = db.get(key);
                    v.timeout = KEY_TIMEOUT;
                    memcache.set(key,value,KEY_TIMEOUT*2);
                    memcache.delete(key_mutex);
                }else{
                    sleep(50);
                    retry();
                }
            }
        }
    }
    
    
    ```

17.  join, left join 区别，左连接原理，右连接，内连接，外连接

18. 数据库查询慢怎么解决，

19. 最左匹配原则，为什么是最左匹配原则？

20. 

    

    



## 操作系统

1. 进程线程的区别
2. linux常用指令  会不会awk 全文替换 统计
3. linux查看内存指令，修改权限指令，如果权限相关的数字是777，代表什么含义（1，2，4的二进制相加）
4. 硬连接、软连接
5. 进程、线程的通信
6. Linux下运行的python程序是进程还是线程？
7. cpu对进程的调度
8. Linux下的中断
9. Linux下的命令？Grep？
10. 如何实现非的查找？进程的查看命令？如何只查看java相关的进程命令
11. Linux下的管道  
12. 操作系统进程调度策略

## 计算机网络

1. 了解TCP不，TCP断开连接的过程说一下（四次挥手）

2. 知道time_wait吗？为啥要有这个状态

3. Ip寻址流程

4.  select poll epoll

5. TCP中的拥塞控制，流量控制

6. 浏览器输入url之后发生了什么

7. 服务器端出现大量的TIME_WAIT连接的原因

   ```
   TIME_WAIT是主动关闭连接的一方保持的状态,在发起主动关闭连接后,发送完最后一次ACK包之后,就会进入到这个状态,然后在保持这个状态2MSL(max segment lifetime) 时间之后,彻底关闭回收资源。  
   
      为什么要保持这一段时间的资源呢?原因有下面两个方面  
   
   	防止上一次连接中的包,迷路后再次出现,影响新的连接(经过2msl,上一次连接的所有的重复包都会消失)    
   可靠的关闭tcp连接。在主动关闭方发送最后一个ack(fin)是有可能丢失的,这个时候被动方会重新发送fin,这个时候如果主动方处于CLOSED状态,就会响应rst而不是ack。所以主动方要处于TIME_WAIT状态而不是CLOSED。另外这么设计TIME_WAIT会定时的回收资源,并不会占用很大资源的,除非短时间内接受大量请求或者受到攻击。    
   
      解决思路:快速回收和重新用哪些TIME_WAIT的资源  
   
      应用层面:  避免频繁关闭连接,如业务优化或者使用长连接等.  
   
      系统层面: 1.缩短msl时间 2.增加可用端口的数量  
   
      msl时间修改:   默认为2分钟  
   
      查看：sysctl -a | grep time | grep wait   
   
   vi /etc/sysctl.conf    
   net.ipv4.tcp_fin_timeout = 30    
   执行 /sbin/sysctl -p让参数生效       进行一些参数的配置  
   ```

   

8. 常见的请求方法getpost putdelete post与put的本质区别。幂等性

9. http, tcp区别，TCP的特点，HTTP与TCP分别在什么层

10. tcp三次握手、第一次握手发送了哪些信息

11. HTTPS与HTTP的差别，

12. http中的session、cookie

    

## 算法题

1. 给一个数组判断是不是二分搜索数的先序遍历。
2. 链表翻转、判断一个链表是否存在环
3. 最长递增子序列
4. 常用的排序算法，提问了时间复杂度和快排的过程
5. 最长公共子串
6. 寿司算法
7. 浮点数的平方根
8. 归并排序
9. 三数之和
10. 快排