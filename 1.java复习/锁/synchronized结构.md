#synchronized


Synchronized 核心组件 
1) Wait Set：哪些调用 wait 方法被阻塞的线程被放置在这里； 
2) Contention List：竞争队列，所有请求锁的线程首先被放在这个竞争队列中； 
3) Entry List：Contention List 中那些有资格成为候选资源的线程被移动到 Entry List 中； 
4) OnDeck：任意时刻，最多只有一个线程正在竞争锁资源，该线程被成为 OnDeck； 
5) Owner：当前已经获取到所资源的线程被称为 Owner； 
6) !Owner：当前释放锁的线程。



处于 ContentionList、EntryList、WaitSet 中的线程都处于阻塞状态，该阻塞是由操作系统 来完成的（Linux 内核下采用 pthread_mutex_lock 内核函数实现的）