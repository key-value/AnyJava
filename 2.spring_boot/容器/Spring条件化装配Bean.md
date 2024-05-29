Spring4 引入了@Conditional 注解，可配合@Bean 或者@Component 注解一起使用。
用 CET4 考试来演示@Conditional 注解。

新建考试结果 Result 接口：

```
public interface Result {
    public void getResult();
}
```

实现类 CET4：

```
public class CET4 implements Result{
	public void getResult() {
		 System.out.println("恭喜你通过CET4");
		   }
	 }
```

定义学生接口：

```
public interface Student {
	public void exam();
	}
```

实现类 XiaoMing：

```
public class XiaoMing implements Student{
	//注入result
	@Autowired(required=false)
	private Result result;
	public void exam() {
		if(result == null)
			System.out.println("抱歉，CET4未通过");
		else
			result.getResult();
	}
}
```

配置 JavaConfig：

```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ConditionConfig {
    @Bean
    @Conditional(ResultCondition.class)
    public Result result(){
        return new CET4();
    }
    @Bean(name="xiaoming")
    public Student student(){
        return new XiaoMing();
    }
}
```

可以看到，@Conditional 中给定了一个 Class，它指明了条件——在本例中，也就是 ResultCondition：


```
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class ResultCondition implements Condition{

    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment env = context.getEnvironment();
        //获取环境变量中的result属性
        String result = env.getProperty("result");
        if("success".equals(result)) return true;
        else return false;
    }

}
```


设置给`@Conditional`的类可以是任意实现了 Condition 接口的类型。可以看出来，这个接口实现起来很简单直接，只需提供 matches()方法的实现即可。如果 matches()方法返回 true，那么就会创建带有@Conditional 注解的 bean。如果 matches()方法返回 false，将不会创建这些 bean。

ConditionContext 是一个接口，大致如下所示：

```
public interface conditioncontext{
	BeanDefinitionRegistry getRegistry();
	ConfigurableListableBeanFactory getBeanFactory();
	ResourceLoader getResourceLoader();
	Environment getEnvironment();
	ClassLoder getClassLoader();
}
```


通过 ConditionContext，我们可以做到如下几点：

1.借助 getRegistry()返回的 BeanDefinitionRegistry 检查 bean 定义；

2.借助 getBeanFactory()返回的 ConfigurableListableBeanFactory 检查 bean 是否存在，甚至探查 bean 的属性；

3.借助 getEnvironment()返回的 Environment 检查环境变量是否存在以及它的值是什么；

4.读取并探查 getResourceLoader()返回的 ResourceLoader 所加载的资源；

5.借助 getClassLoader()返回的 ClassLoader 加载并检查类是否存在。

AnnotatedTypeMetadata 则能够让我们检查带有@Bean 注解的方法上还有什么其他的注解。像 ConditionContext 一样，AnnotatedTypeMetadata 也是一个接口。它如下所示：

```
public interface AnnotatedTypeMetadata{
	boolean isAnnotated(String annotationType);
	Map<String,Object> getAnnotationAttributes(String annotationType);
	Map<String,Object> getAnnotationAttributes(
	String annotationType,boolean classValuesAsString);
	MutilValueMap<String,Object> getAllAnnotationAttributes(
	String annotationType);
	MutilValueMap<String,Object> getAllAnnotationAttributes(
	String annotationType,boolean classValuesAsString);
}
```


借助 isAnnotated()方法，我们能够判断带有@Bean 注解的方法是不是还有其他特定的注解。借助其他的那些方法，我们能够检查@Bean 注解的方法上其他注解的属性。

现在测试@Conditional 注解的作用：

```
public class TestConditional {
	public static void main(String[] args) {
		System.setProperty("result", "fail");
		ApplicationContext ac = new AnnotationConfigApplicationContext(ConditionConfig.class);  
		Student xiaoming = (Student) ac.getBean("xiaoming");
		xiaoming.exam();
	}
}
```


这里设置 result 为 fail，ResultCondition 的 matches()方法返回 false，所以 CET4 Bean 并不会被创建，结果应该输出“抱歉，CET4 未通过”，测试结果：


```
 抱歉，CET4 未通过 
```

