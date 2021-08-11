# String

## 不可变性
*.字符串常量池的需要
*.允许 String 对象缓存 HashCode
*.String 被许多的 Java 类(库)用来当做参数，例如：网络连接地址 URL、文件路径 path、还有反射机制所需要的 String 参数等, 假若 String 不是固定不变的，将会引起各种安全隐患

## 内部实现
`String` 类中使用 final 关键字修饰字符数组来保存字符串，`private final char value[]`，所以`String` 对象是不可变的

## StringBuild和StringBuffer
`StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串`char[]value` 但是没有用 `final` 关键字修饰，所以这两种对象都是可变的。


## 线程安全
`String` 中的对象是不可变的，也就可以理解为常量，线程安全。
`AbstractStringBuilder` 是 `StringBuilder` 与 `StringBuffer` 的公共父类，定义了一些字符串的基本操作，如 `expandCapacity`、`append`、`insert`、`indexOf` 等公共方法。`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。

>StringBuffer 中并不是所有方法都使用了 Synchronized 修饰来实现同步：
```
@Override
public StringBuffer insert(int dstOffset, CharSequence s) {  
    // Note, synchronization achieved via invocations of other StringBuffer methods  
    // after narrowing of s to specific type  
    // Ditto for toStringCache clearing  
    super.insert(dstOffset, s);  
    return this;
}
```

执行效率：StringBuilder > StringBuffer > String


# String 字符串修改实现的原理？
当用 String 类型来对字符串进行修改时，其实现方法是首先创建一个 StringBuffer，其次调用 StringBuffer 的 append() 方法，最后调用 StringBuffer 的 toString() 方法把结果返回。