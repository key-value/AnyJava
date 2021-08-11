![[Pasted image 20210729133133.png]]


![[Pasted image 20210729170142.png]]


AQS 核心思想是：如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中

>CLH队列：CLH(Craig, Landin, and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配。

![[Pasted image 20210803115900.png]]

实现
1. ReentrantLock
2. ReentrantReadWriteLock

[源码解析](https://zhuanlan.zhihu.com/p/265096774)


# AQS 对资源的共享模式有哪些？

1.  Exclusive（独占）：只有一个线程能执行，如：ReentrantLock，又可分为公平锁和非公平锁：
    
2.  Share（共享）：多个线程可同时执行，如：CountDownLatch、Semaphore、CountDownLatch、 CyclicBarrier、ReadWriteLock。