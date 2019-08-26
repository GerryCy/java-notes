# [JUC知识](#JUC知识)

- [StampedLock](#StampedLock使用)
  - [StampedLock使用](#StampedLock使用)
  - [StampedLock原理](#StampedLock原理)
- [Condition](#condition)
  - [Condition使用](#Condition使用)
  - [Condition原理](#Condition)
- [Semaphore](#semaphore)
  - [Semaphore使用](#Semaphore使用)
  - [Semaphore原理](#Semaphore原理)
- [Exchanger](#exchanger)
- [ExecutorService](#executorservice)
- [Phaser](#phaser)
- [ForkJoin](#forkjoin)
- [ConcurrentHashMap](#concurrenthashmap)
- [ConcurrentLinkedDeque](#concurrentlinkeddeque)
- [ConcurrentSkipListMap](#concurrentskiplistmap)
- [ConcurrentSkipSet](#concurrentskipset)
- [CopyOnWriteArrayList](#copyonwritearraylist)
- [CopyOnWriteArraySet](#copyonwritearrayset)
- [DelayQueue](#delayqueue)
- [LinkedBlockingDeque](#linkedblockingdeque)
- [LinkedBlockingQueue](#linkedblockingqueue)
- [LinkedTransferQueue](#linkedtransferqueue)
- [PriorityBlockingQueue](#priorityblockingqueue)
- [CompletableFuture](#completablefuture)
- [自定义ThreadPoolExecutor](#自定义threadpoolexecutor)
- [优先级线程池](#优先级线程池)
- [ThreadFactory](#threadfactory)
- [自定义Lock](#自定义lock)
- [自定义原子对象](#自定义原子对象)

### 

## StampedLock

### StampedLock使用

### StampedLock原理

## Condition

### Condition使用

### Condition原理

## CountDownLatch

### CountDownLatch使用

### CountDownLatch原理

![countdownlatch-class](../../assert/countdownlatch-class.png)

```java
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;

    Sync(int count) {
        setState(count);
    }

    int getCount() {
        return getState();
    }

    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```



## CylicBarrier

### CylicBarrier使用

### CylicBarrier原理

## Semaphore

## Exchanger

## ExecutorService

## Phaser

## ForkJoin

## ConcurrentHashMap

## ConcurrentLinkedDeque

## ConcurrentSkipListMap

## ConcurrentSkipSet

## CopyOnWriteArrayList

## CopyOnWriteArraySet

## DelayQueue

## LinkedBlockingDeque

## LinkedBlockingQueue

## LinkedTransferQueue

## PriorityBlockingQueue

## CompletableFuture

## 自定义ThreadPoolExecutor

## 优先级线程池

## ThreadFactory

## 自定义Lock

## 自定义原子对象







