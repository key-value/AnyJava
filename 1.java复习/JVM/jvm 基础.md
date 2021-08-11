
1.  类加载器（ClassLoader）
    
2.  运行时数据区（Runtime Data Area）
    
3.  执行引擎（Execution Engine）
    
4.  本地库接口（Native Interface）

各组件的作用：首先通过类加载器（ClassLoader）会把 Java 代码转换成字节码，运行时数据区（Runtime Data Area）再把字节码加载到内存中，而字节码文件只是 JVM 的一套指令集规范，并不能直接交给底层操作系统去执行，因此需要特定的命令解析器执行引擎（Execution Engine），将字节码翻译成底层系统指令，再交由 CPU 去执行，而这个过程中需要调用其他语言的本地库接口（Native Interface）来实现整个程序的功能。

![[Pasted image 20210717132401.png]]

# 类加载机制
1. 加载
2. 验证
3. 准备
4. 解析
5. 初始化

