本文大纲如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEQnkSSx5R6Wrt32TF9Mw1fLNjlU4scpc8STbdwLHOoZHzUVhA8Jcq2w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 纵观全局

我的英文名叫`ReentrantReadWriteLock`（后面简称`RRW`），大家喜欢叫我读写锁，因为我常年混迹在读多写少的场景。

## 读写锁规范

作为合格的读写锁，先要有读锁与写锁才行。

所以声明了`ReadWriteLock`接口，作为读写锁的基本规范。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEvB6yiayChV4agfVVztad6szuuVMd9MuDCvqTibeqLDUscUkA6Uyy38hQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

之后都是**围绕着规范**去实现读锁与写锁。

## 读锁与写锁

WriteLock与ReadLock就是读锁和写锁，它们是RRW实现`ReadWriteLock`接口的产物。

但读锁、写锁也要遵守锁操作的基本规范。

所以WriteLock与ReadLock都实现了`Lock`接口。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEtibuMJF5Oe0wdPRrXRiaL6ef9gP5UWpD4GHicN9piadicibIcwjiaJh7v5EJw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

那么WriteLock与ReadLock对Lock接口具体是**如何实现**的呢？

自然是少不了我们的老朋友`AQS`了。

## AQS

众所周知，要实现锁的基本操作，必须要仰仗`AQS`老大哥了。

`AQS（AbstractQueuedSynchronizer）`抽象类定义了一套多线程访问共享资源的同步模板，解决了实现同步器时涉及的大量细节问题，能够极大地减少实现工作，用大白话来说，`AQS`为加锁和解锁过程提供了统一的模板函数，只有少量细节由子类自己决定。

AQS简化流程图如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEia0GicHkT1IKRR3eJiakxgg2vSL964MB3Wgn0B61MiblUJM5iaya1P4s9Uw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## Sync

AQS为加锁和解锁过程提供了统一的模板函数，只有少量细节由子类自己决定，但是WriteLock与ReadLock**没有直接去继承AQS**。

因为WriteLock与ReadLock觉得，自己还要去继承`AQS`实现一些两者可以公用的抽象函数，不仅麻烦，还有重复劳动。

所以干脆单独提供一个对锁操作的类，由WriteLock与ReadLock持有使用，这个类叫`Sync`。

Sync继承AQS实现了如下的核心抽象函数

- **tryAcquire**
    
- **release**
    
- **tryAcquireShared**
    
- **tryReleaseShared**
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEEYUv6siaylXRfQ7CIu1mEn5q9zQoiaKic9jHZsluvkBkpPhic7slj0Ctkw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中tryAcquire、release是为`WriteLock`写锁准备的。

tryAcquireShared、tryReleaseShared是为`ReadLock`读锁准备的，这里后面会说。

上面说了Sync实现了一些`AQS`的核心抽象函数，但是Sync本身也有一些重要的内容，看看下面这段代码

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WERgqHHzlKb5MJGtyiaq1IIiaCAYQG98riczOJ0NaNH58qia4Zz82Zqwgx5Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们都知道`AQS`中维护了一个`state`状态变量，正常来说，维护读锁与写锁状态需要两个变量，但是为了节约资源，使用高低位切割实现`state`状态变量维护两种状态，即高`16`位表示读状态，低`16`位表示写状态。

Sync中还定义了HoldCounter与ThreadLocalHoldCounter

- **HoldCounter是用来记录读锁重入数的对象**
    
- **ThreadLocalHoldCounter是ThreadLocal变量，用来存放第一个获取读锁线程外的其他线程的读锁重入数对象**
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEvfGqGNRlQn60kzwvXbicFUEdQlaLibQqUgcnqt9oyTe0PgYcYnn6cwZQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 公平与非公平策略

你看，人家`ReentrantLock`都有公平与非公平策略，所以`ReentrantReadWriteLock`也要有。

什么是公平与非公平策略？

因为在`AQS`流程中，获取锁失败的线程，会被构建成节点入队到`CLH`队列，其他线程释放锁会唤醒`CLH`队列的线程重新竞争锁，如下图所示（简化流程）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WE6u6sv5qicVbCsibHDiazPfjRKrvwiajnWfcWUGQib5fvjBpibVhYIUydT3Cg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

非公平策略是指，非`CLH`队列的线程与`CLH`队列的线程竞争锁，大家各凭本事，不会因为你是`CLH`队列的线程，排了很久的队，就把锁让给你。

公平策略是指，严格按照`CLH`队列顺序获取锁，一定会让`CLH`队列线程竞争成功，如果非`CLH`队列线程一直占用时间片，那就一直失败，直到时间片轮到`CLH`队列线程为止，所以公平策略的性能会更差。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEicUIHZTrHoOG8Iuw0HmLfeMfba63OXIRWUKz5mGJlnNLkKUawfS1TqQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

回到正题，为了支持公平与非公平策略，Sync扩展了`FairSync、NonfairSync`子类，两个子类实现了readerShouldBlock、writerShouldBlock函数，**即读锁与写锁是否阻塞**。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEnybialXJNG9GZxRNfnFwiavhb0P8w8Td8OicoRGQznkuFkHMut4NjfMbA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

readerShouldBlock、writerShouldBlock函数在什么地方使用后面会说。

## ReentrantReadWriteLock全局图

最后把前面讲过的内容，全部组装起来，构成下面这张图。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEsmLAYNhasbrSicOiaCesN4meZib3YpXX80BzhGh0Ueg5wraviaAppVqn4Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

有了全局观后，后面就可以深入细节逐个击破了。

# 深入细节

后面我们只要攻破`5`个细节就够了，分别是读写锁的创建、获取写锁、释放写锁、获取读锁、释放读锁。

## ReentrantReadWriteLock的创建

读写锁的创建，会初始化化一系列类，代码如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEGteXqjcTGG7r8pJ2Q6IUU7DMU0WgeHDNjyAjgE0iclTKMMicHtdNfGKQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`ReentrantReadWriteLock`默认是非公平策略，如果想用公平策略，可以直接调用有参构造器，传入`true`即可。

但不管是创建FairSync还是NonfairSync，都会触发`Sync`的无参构造器，因为`Sync`是它们的父类（**本质上它们俩都是Sync**）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEzH2ZRR39GzZJxZrcGic2O71Xb3g5VjfFTcLU2rI80m7deBpnCE5VnYA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

因为Sync需要提供给ReadLock与WriteLock使用，所以创建ReadLock与WriteLock时，会接收`ReentrantReadWriteLock`对象作为入参。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEHjzj1lQlyoa9yGFSaVAZZf6pM0d7wbzuwAicDZ8naIz7XmZy9dk8wbQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

最后通过`ReentrantReadWriteLock.sync`把`Sync`交给了ReadLock与WriteLock。

## 获取写锁

我们遵守ReadWriteLock接口规范，调用`ReentrantReadWriteLock.writeLock`函数获取写锁对象。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WE2tA7IUUkjV4yhhAaeV0wvokWEPC7MUZV2jfJS0Z0EzQmnQr2BZlLoQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

获取到写锁对象后，遵守Lock接口规范，调用`lock`函数获取写锁。

WriteLock.lock函数是由`Sync`实现的（**FairSync或NonfairSync**）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WE9qr97tYzS0gdEPcicricCTuyauPGMZo3OBiaaq9WicnlyduRiaU87IVEkTw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`sync.acquire(1)`函数是AQS中的独占式获取锁流程模板（Sync继承自AQS）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WE3Ijb1gV1y8sYBAVDswxczICiag1N9Ghbjr8icsaJmj6Yibssvt0eFCOeg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

WriteLock.lock调用链如下图

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEDZw2LNyaRuJo1BWnjA6ucAd76iclumfq80SfpTZrcp3pbT2TfB0p4eA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们只关注`tryAcquire`函数，其他函数是AQS的获取独占式锁失败后的流程内容，不属于本文范畴，`tryAcquire`函数代码如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEZjuBic5R7LydXnhngvIeqz36ZKibSgcoQXSKVOKpt0dsHWGebawtfkFQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

为了易于理解，我把它转成流程图

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEgDGxYOXo1HL5E7Lp59richhg0K6Z8Hgx2qWyCVrKoKAe78zeuAg3NwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过流程图，我们发现了一些要点

- **读写互斥**
    
- **写写互斥**
    
- **写锁支持同一个线程重入**
    
- **writerShouldBlock写锁是否阻塞实现取决公平与非公平的策略（FairSync和NonfairSync）**
    

## 释放写锁

获取到写锁，临界区执行完，要记得释放写锁（**如果重入多次要释放对应的次数**），不然会阻塞其他线程的读写操作，调用`unlock`函数释放写锁（**Lock接口规范**）。

WriteLock.unlock函数也是由Sync实现的（**FairSync或NonfairSync**）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEIsBzVP6ODiaBYIBeNOkh7zYy89Fmiaf9EqakLueN5mOEKG3XYicts0viaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`sync.release(1)`执行的是AQS中的独占式释放锁流程模板（Sync继承自AQS）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WE7Q8BqibqONOU6KkMI18aRukBkv4kGyjDkNgUYEjF7ke7k4mNGVuhTYQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

WriteLock.unlock调用链如下图

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEXWGIb0YgQK7q06yp0IKr63kw64ZqPozeHBa369Mxt7pLLItPBVYbWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

再来看看`tryRelease`函数，其他函数是AQS的释放独占式成功后的流程内容，不属于本文范畴，`tryRelease`函数代码如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEhJ9XH9fMDhibcib2cYibqUm2Ctoz8LqthxN3t1M5q3XHuY8oicZyibTuzrg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

为了易于理解，我把它转成流程图

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEuImClQXRPGHDrFibJ8tJic8ae5UtejOOtZBPvhIQs96v7dEXyh7Ma0ug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

因为同一个线程可以对相同的写锁重入多次，所以也要释放的相同的次数。

## 获取读锁

我们遵守ReadWriteLock接口规范，调用`ReentrantReadWriteLock.readLock`函数获取读锁对象。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEoiaic1QdA0TsCcyqw84FKEZf2KpqLCIRWeGPo51E0ZqG6DmxJiazJKwmQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

获取到读锁对象后，遵守Lock接口规范，调用`lock`函数获取读锁。

ReadLock.lock函数是由`Sync`实现的（**FairSync或NonfairSync**）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WE3TdWMickOkVSF1X1IeRfRLP5Ajd8FaSY1RDBWBRlBwtuPTibbeVaudRA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`sync.acquireShared(1)`函数执行的是AQS中的共享式获取锁流程模板（Sync继承自AQS）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEoWrYibb1gzplMhOfUIFUzUGDZknWkGb6qeksztm9tlY6LwQpSW1lpJw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

ReadLock.lock调用链如下图

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEtVw0L8qx3GibEzsXsiaGr82oB7I9ibzKeRJj08v7Tt2OlDEKDAKZeDFAQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们只关注`tryAcquireShared`函数，doAcquireShared函数是AQS的获取共享式锁失败后的流程内容，不属于本文范畴，`tryAcquireShared`函数代码如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEDo9GibZUBOxheKVA8HhmAWohYYLQiaykOeicNSbHnFTOqHRCboSMzkbYg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

代码还挺多的，为了易于理解，我把它转成流程图

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEe66icjxm8IXGsxd9QXgDQCfG7CUOnauAC2t17vQaKDUXiavrzUueU3TA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过流程图，我们发现了一些要点

- **读锁共享，读读不互斥**
    
- **读锁可重入，每个获取读锁的线程都会记录对应的重入数**
    
- **读写互斥，锁降级场景除外**
    
- **支持锁降级，持有写锁的线程，可以获取读锁，但是后续要记得把读锁和写锁读释放**
    
- **readerShouldBlock读锁是否阻塞实现取决公平与非公平的策略（FairSync和NonfairSync）**
    

## 释放读锁

获取到读锁，执行完临界区后，要记得释放读锁（**如果重入多次要释放对应的次数**），不然会阻塞其他线程的写操作，通过调用`unlock`函数释放读锁（**Lock接口规范**）。

ReadLock.unlock函数也是由Sync实现的（**FairSync或NonfairSync**）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEkjxPL9PtuiakTBJsWKJZpibUL2ywba09e3oU4UekenZFWIibgf38H4vqQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`sync.releaseShared(1)`函数执行的是AQS中的共享式释放锁流程模板（Sync继承自AQS）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEqJdZcYkBpyvY5dickN3ibqoFUPN18vzcicVI7x7w7FQxnqVJfoQy0RgQw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

ReadLock.unlock调用链如下图

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEDshxiboiarUMDjOY0WNzhMh8PLmld3XabqXyBxDDO0bUXGqoDmq31BnA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们只关注`tryReleaseShared`函数，doReleaseShared函数是AQS的释放共享式锁成功后的流程内容，不属于本文范畴，`tryReleaseShared`函数代码如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEdb9Ns8KJaSB4M5SviatmSziaUjWOKibya8jQCK98KK1k7qcTrpt46kh8w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

为了易于理解，我把它转成流程图

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nx2zmNic4YYIJgbIvjyWI9WEXZsxxWmYXicavMwEF1q95Mhwa7CRHia136xXLvQDBHicyB8X0hJ9xXTOw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里有三点需要注意

- **第一点：线程读锁的重入数与读锁数量是两个概念，线程读锁的重入数是每个线程获取同一个读锁的次数，读锁数量则是所有线程的读锁重入数总和。**
    
- **第二点：AQS的共享式释放锁流程模板中，只有全部的读锁被释放了，才会去执行doReleaseShared函数**
    
- **第三点：因为使用的是AQS共享式流程模板，如果CLH队列后面的线程节点都是因写锁阻塞的读锁线程节点，会传播唤醒**
    

# 小结

最后做个小结，`ReentrantReadWriteLock`底层实现与`ReentrantLock`思路一致，它们都离不开`AQS`，都是声明一个继承`AQS`的`Sync`，并在`Sync`下扩展公平与非公平策略，后续的锁相关操作都委托给公平与非公平策略执行。

我们还发现，在`AQS`中除了独占式模板，还有共享式模板，它们在多线程访问共享资源的流程会有所差异，就如`ReentrantReadWriteLock`中读锁使用共享式，写锁使用独占式。

最后再捋一捋写锁与读锁的逻辑

1. 读读不阻塞
    
2. 写锁阻塞写之后的读写锁，但是不阻塞写锁之前的读锁线程
    
3. 写锁会被写之前的读写锁阻塞
    
4. 读锁节点唤醒会无条件传播唤醒CLH队列后面的读锁节点
    
5. 写锁可以降级为读锁，防止更新丢失
    
6. 读锁、写锁都支持重入