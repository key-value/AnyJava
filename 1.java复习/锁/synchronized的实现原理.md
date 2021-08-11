 synchronized的实现原理
 
 
 #synchronized 
 Java 虚拟机中的同步(Synchronization)**基于进入和退出管程(Monitor)对象实现**
 
 同步代码块是使用monitorenter和monitorexit来实现的，同步方法 并不是由 monitor enter 和 monitor exit 指令来实现同步的，而是由方法调用指令读取运行时常量池中方法的 **ACC_SYNCHRONIZED** 标志来隐式实现的。monitorenter指令是在编译后插入同步代码块的起始位置，而monitorexit指令是在方法结束处和异常处，每个对象都有一个monitor与之关联，当一个monitor被持有后它就会处于锁定状态。
 
 
 synchronized用的锁是存在**Java对象头**
 
 


[引用](https://blog.csdn.net/fighting_sxw/article/details/79971253)
[引用](https://www.cnblogs.com/fsmly/p/10703804.html)