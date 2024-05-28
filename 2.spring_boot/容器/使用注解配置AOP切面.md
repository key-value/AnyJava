## 注解切面

除了使用XML配置AOP切面，我们还可以使用更简洁的注解配置。

现使用注解修改Audience类：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24  <br>25  <br>26  <br>27  <br>28  <br>29  <br>30  <br>31  <br>32  <br>33  <br>34  <br>35  <br>36|import org.aspectj.lang.ProceedingJoinPoint;  <br>import org.aspectj.lang.annotation.AfterReturning;  <br>import org.aspectj.lang.annotation.AfterThrowing;  <br>import org.aspectj.lang.annotation.Aspect;  <br>import org.aspectj.lang.annotation.Before;  <br>import org.aspectj.lang.annotation.Pointcut;  <br>import org.springframework.stereotype.Component;  <br>@Component  <br>@Aspect  <br>public class Audience {  <br>    // 声明切点  <br>    @Pointcut("execution(* com.spring.entity.Performer.perform(..))")  <br>    public void performance(){  <br>    	  <br>    }  <br>    // 表演之前  <br>    @Before("performance()")  <br>    public void takeSeats(){  <br>    	System.out.println("观众入座");  <br>    }  <br>    // 表演之前  <br>    @Before("performance()")  <br>    public void turnOffCellPhones(){  <br>    	System.out.println("关闭手机");  <br>    }  <br>    // 表演之后  <br>    @AfterReturning("performance()")  <br>    public void applaud(){  <br>    	System.out.println("啪啪啪啪啪");  <br>    }  <br>    // 表演失败后  <br>    @AfterThrowing("performance()")  <br>    public void failure(){  <br>    	System.out.println("坑爹，退钱！");  <br>    }  <br>}|

`@Aspect`使得Audience成为了切面。

为了让Spring识别改注解，我们还需在XML中添加`<aop:aspectj-autoproxy/>`。`<aop:aspectj-autoproxy/>`将在Spring应用上下文中创建一个`AnnotationAwareAspectJAutoProxyCreator`类，它会自动代理`@Aspect`标注的Bean：

|   |   |
|---|---|
|1|<aop:aspectj-autoproxy proxy-target-class="true" />|

实例化kenny，输出：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6|观众入座  <br>关闭手机  <br>唱：May Rain  <br>吹萨克斯  <br>啪啪啪啪啪  <br>和XML配置AOP效果一样。|

在这途中遇到错误：

|   |   |
|---|---|
|1|：error at ::0 can't find referenced pointcut XXX|

网络上的说法是JDK不匹配造成的，

我原来用的JDK1.7匹配的是aspectjrt.1.6和aspectjweaver.1.6,因此会报错。 如果要使用AspectJ完成注解切面需要注意下面的JDK与AspectJ的匹配：

JDK1.6 —— aspectJ1.6

JDK1.7 —— aspectJ1.7.3+

> aspectJ1.7下载链接：[密码：6wd7](http://pan.baidu.com/s/1pLrlXTD)

## [](https://mrbird.cc/%E4%BD%BF%E7%94%A8%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AEAOP%E5%88%87%E9%9D%A2.html#%E6%B3%A8%E8%A7%A3%E7%8E%AF%E7%BB%95%E9%80%9A%E7%9F%A5 "注解环绕通知")注解环绕通知

使用@Around注解环绕通知：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24|@Component  <br>@Aspect  <br>public class Audience {  <br>    // 声明切点  <br>    @Pointcut("execution(* com.spring.entity.Performer.perform(..))")  <br>    public void performance(){  <br>    	  <br>    }  <br>    @Around("performance()")  <br>    public void watch(ProceedingJoinPoint joinpoint){  <br>        try{  <br>            System.out.println("观众入座");  <br>            System.out.println("关闭手机");  <br>            long start=System.currentTimeMillis();  <br>            // 执行被通知的方法！  <br>            joinpoint.proceed();  <br>            long end=System.currentTimeMillis();  <br>            System.out.println("啪啪啪啪啪");  <br>            System.out.println("表演耗时："+(end-start)+" milliseconds");  <br>        }catch(Throwable t){  <br>            System.out.println("坑爹，退钱！");  <br>        }  <br>    }  <br>}|

实例化kenny，输出：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6|观众入座  <br>关闭手机  <br>唱：May Rain  <br>吹萨克斯  <br>啪啪啪啪啪  <br>表演耗时：30 milliseconds|

## [](https://mrbird.cc/%E4%BD%BF%E7%94%A8%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AEAOP%E5%88%87%E9%9D%A2.html#%E6%B3%A8%E8%A7%A3%E4%BC%A0%E9%80%92%E5%8F%82%E6%95%B0 "注解传递参数")注解传递参数

修改Magician类：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24|import org.aspectj.lang.annotation.Aspect;  <br>import org.aspectj.lang.annotation.Before;  <br>import org.aspectj.lang.annotation.Pointcut;  <br>import org.springframework.stereotype.Component;  <br>@Component  <br>@Aspect  <br>public class Magician implements MindReader{  <br>    private String thoughts;  <br>    // 声明参数化切点  <br>    @Pointcut("execution(* com.spring.entity."  <br>    	      + "Thinker.thinkOfSomething(String)) && args(thoughts)")  <br>    public void thinking(String thoughts) {  <br>    	  <br>    }  <br>    // 把参数传递给通知  <br>    @Before("thinking(thoughts)")  <br>    public void interceptThoughts(String thoughts) {  <br>    	System.out.println("侦听志愿者的心声");  <br>    	this.thoughts=thoughts;  <br>    }	  <br>    public String getThoughts() {  <br>    	return thoughts;  <br>    }  <br>}|

`<aop:pointcut>`变为`@Pointcut`注解，`<aop:before>`变为`@Before`注解。

测试：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10|public class TestIntercept {  <br>    public static void main(String[] args) {  <br>        ApplicationContext ac  <br>            = new ClassPathXmlApplicationContext("applicationContext.xml");  <br>        Volunteer volunteer = (Volunteer) ac.getBean("volunteer");  <br>        volunteer.thinkOfSomething("演出真精彩！");  <br>        Magician magician = (Magician) ac.getBean("magician");  <br>        System.out.println("志愿者心里想的是："+magician.getThoughts());  <br>    }  <br>}|

输出：

|   |   |
|---|---|
|1  <br>2|侦听志愿者的心声  <br>志愿者心里想的是：演出真精彩！|

## [](https://mrbird.cc/%E4%BD%BF%E7%94%A8%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AEAOP%E5%88%87%E9%9D%A2.html#%E9%80%9A%E8%BF%87%E6%B3%A8%E8%A7%A3%E5%BC%95%E5%85%A5%E6%96%B0%E7%9A%84%E6%96%B9%E6%B3%95 "通过注解引入新的方法")通过注解引入新的方法

之前通过在XML中配置AOP切面的方法为Bean引入新的方法，现在改用注解的方式来实现：

新建一个ContestantIntroducer类：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12|import org.aspectj.lang.annotation.Aspect;  <br>import org.aspectj.lang.annotation.DeclareParents;  <br>import org.springframework.stereotype.Component;  <br>@Component  <br>@Aspect  <br>public class ContestantIntroducer {  <br>   <br>    @DeclareParents(  <br>      value = "com.spring.entity.Performer+",   <br>      defaultImpl = OutstandingContestant.class)  <br>    public static Contestant contestant;  <br>}|

`@DeclareParents`注解代替了之前的`<aop:declare-parents>`标签。 `@DeclareParents`注解由三个部分组成：

1.value属性等同于`<aop:declare-parents>`的types-matching属性。它标识应该被引入指定接口的Bean。

2.defaultImpl属性等同于`<aop:declare-parents>`的default-impl属性。它标识该类所引入接口的实现。

3.由`@DeclareParents`注解所标注的static属性制订了将被引入的接口。

测试：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10|public class Play {  <br>    public static void main(String[] args) {  <br>        ApplicationContext ac  <br>            = new ClassPathXmlApplicationContext("applicationContext.xml");  <br>        Instrumentalist kenny = (Instrumentalist)ac.getBean("kenny");  <br>        kenny.perform();  <br>        Contestant kenny1=(Contestant) ac.getBean("kenny");  <br>        kenny1.receiveAward();  <br>    }  <br>}|

输出：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7|观众入座  <br>关闭手机  <br>唱：May Rain  <br>吹萨克斯  <br>啪啪啪啪啪  <br>表演耗时：27 milliseconds  <br>参加颁奖典礼|




https://mrbird.cc/%E4%BD%BF%E7%94%A8%E6%B3%A8%E8%A7%A3%E9%85%8D%E7%BD%AEAOP%E5%88%87%E9%9D%A2.html