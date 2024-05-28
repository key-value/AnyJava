传统的Java应用，Bean的生命周期很简单。使用Java关键字new进行Bean的实例化，然后该Bean就可以使用了。一旦该Bean不再被使用，则有GC选择回收。

相比之下，在Spring容器中，Bean的生命周期要细腻的多，大致过程如下图所示：

![14194492-file_1487994485619_4dfe.png](https://mrbird.cc/img/14194492-file_1487994485619_4dfe.png)

1.Spring对Bean进行实例化；

2.Spring将值和对Bean的引用注入进Bean对应的属性中；

3.如果Bean实现了BeanNameAware接口，Spring将Bean的ID传递给setBeanName()接口方法；

4.如果Bean实现了BeanFactoryAware接口，Spring将调用setBeanFactory()接口方法，将BeanFactory容器实例传入；

5.如果Bean实现了ApplicationContextAware接口，Spring将调用setApplicationContext()接口方法，将应用上下文的引用传入；

6.如果Bean实现了BeanPostProcessor接口，Spring将调用它们的postProcessBeforeInitialization()接口方法；

7.如果Bean实现了InitializingBean接口，Spring将调用它们的afterPropertiesSet()接口方法。类似的，如果Bean使用init-method声明了初始化方法，该方法也会被调用。

8.如果Bean实现了BeanPostProcessor接口，Spring将调用它们的postProcessAfterInitialization()方法；

9.此时此刻，Bean已经准备就绪，可以被使用了。它们将一直驻留在Spring容器中，直到容器被销毁；

10.如果Bean实现了DisposableBean接口，Spring将调用它的destroy()接口方法。同样，如果Bean使用destroy-method声明了销毁方法，该方法也会被调用；