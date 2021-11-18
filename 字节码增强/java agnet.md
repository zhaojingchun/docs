随着对 Java 的愈加熟悉，我也了解了反射、字节码等技术，直到前些天的周会分享，有位同事分享了 Btrace 的使用和实现，提到了 Java 的 ASM 框架和 JVM TI 接口。 Btrace 修改代码能力的实现与 Debug 的 Evaluate 有很多相似之处，这大大吸引了我。分享就像一个引子，从中学到的东西只是皮毛，要了解它还是要自己研究。于是自己查看资料并写代码学习了下其具体实现。






https://docs.oracle.com/javase/8/docs/platform/jvmti/jvmti.html

JVM TI（JVM Tool Interface）JVM 工具接口是 JVM 提供的一个非常强大的对 JVM 操作的工具接口，通过这个接口，我们可以实现对 JVM 多种组件的操作，从[JVMTM Tool Interface](https://link.juejin.cn?target=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2F8%2Fdocs%2Fplatform%2Fjvmti%2Fjvmti.html) 这里我们认识到 JVM TI 的强大，它包括了对虚拟机堆内存、类、线程等各个方面的管理接口。


作者：枕边书
链接：https://juejin.cn/post/6844903751082328072
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。





Java AST  语法书



----

抽象语法库

https://github.com/javaparser/javaparser

https://www.bilibili.com/video/BV14q4y1r7GN/?spm_id_from=333.788.recommend_more_video.12



实战：





java -javaagent:java-agent-1.0-SNAPSHOT.jar -cp agent-target-1.0-SNAPSHOT.jar com.jc.Main





内含静态方法premain(),这个方法名是java agent内定的方法名，它总会在main函数之前执行





### addTransformer方法

对 Java 类文件的操作，可以理解为对一个 byte 数组的操作（将类文件的二进制字节流读入一个 byte 数组）。开发者可以在 `interface ClassFileTransformer`的 `transform` 方法（通过 classfileBuffer 参数）中得到，操作并最终返回一个类的定义（一个 byte 数组），接下来演示下`transform` 转换类的用法，采用简单的类文件替换方式。



很好的一片入门文章

https://mp.weixin.qq.com/s?__biz=MzU3MTAzNTMzMQ==&mid=2247484544&idx=1&sn=d279217a75ca58b0d735fb594d423348

我们先忽略上面两个方法的具体玩法，先简单看一下这两个方法的区别，注释上也说了

- jvm 参数形式：调用 premain 方法
- attach 方式：调用 agentmain 方法

其中 jvm 方式，也就是说要使用这个 agent 的目标应用，在启动的时候，需要指定 jvm 参数 `-javaagent:xxx.jar`，当我们提供的 agent 属于基础必备服务时，可以用这种方式

当目标应用程序启动之后，并没有添加`-javaagent`加载我们的 agent，依然希望目标程序使用我们的 agent，这时候就可以使用 attach 方式来使用（后面会介绍具体的使用姿势），自然而然的会想到如果我们的 agent 用来 debug 定位问题，就可以用这种方式



作者：库洛琪
链接：https://www.jianshu.com/p/a1e683227b09
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处

##### 3. Java agent与Instrumentation

Java agent的检测功能是无限的，值得关注的但不仅仅限于这些：

- 运行时重新定义类的能力。重定义将可能改变方法体、常量池和属性。重定义不能增加、删除或者重命名域和方法，不能修改方法的签名、不能改变继承关系。
- 运行时重新转换类的能力。转换将可能改变方法体、常量池和属性。转换不能增加、删除或者重命名域和方法，不能修改方法的签名、不能改变继承关系。

需要注意的是转换和重定义类的字节码是没有验证的，当转换和重定义执行完成后类直接被虚拟机装入了，如果这些字节码是错误的，将会抛出异常并导致JVM崩溃。

**demo**





https://juejin.cn/post/7017781194481729549 很好

Instrumentation接口定义如下：

```
public interface Instrumentation {
    
    //增加一个Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 canRetransform 设置是否允许重新转换。
    void addTransformer(ClassFileTransformer transformer, boolean canRetransform);

    //在类加载之前，重新定义 Class 文件，ClassDefinition 表示对一个类新的定义，如果在类加载之后，需要使用 retransformClasses 方法重新定义。addTransformer方法配置之后，后续的类加载都会被Transformer拦截。对于已经加载过的类，可以执行retransformClasses来重新触发这个Transformer的拦截。类加载的字节码被修改后，除非再次被retransform，否则不会恢复。
    void addTransformer(ClassFileTransformer transformer);

    //删除一个类转换器
    boolean removeTransformer(ClassFileTransformer transformer);

    boolean isRetransformClassesSupported();

    //在类加载之后，重新定义 Class。这个很重要，该方法是1.6 之后加入的，事实上，该方法是 update 了一个类。
    void retransformClasses(Class<?>... classes) throws UnmodifiableClassException;

    boolean isRedefineClassesSupported();

    
    void redefineClasses(ClassDefinition... definitions)
        throws  ClassNotFoundException, UnmodifiableClassException;

    boolean isModifiableClass(Class<?> theClass);

    @SuppressWarnings("rawtypes")
    Class[] getAllLoadedClasses();

  
    @SuppressWarnings("rawtypes")
    Class[] getInitiatedClasses(ClassLoader loader);

    //获取一个对象的大小
    long getObjectSize(Object objectToSize);


   
    void appendToBootstrapClassLoaderSearch(JarFile jarfile);

    
    void appendToSystemClassLoaderSearch(JarFile jarfile);

    
    boolean isNativeMethodPrefixSupported();

    
    void setNativeMethodPrefix(ClassFileTransformer transformer, String prefix);
}

作者：刘p辉
链接：https://juejin.cn/post/7017781194481729549
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```



MANIFREST.MF文件的作用 



```java
public interface ClassFileTransformer {
    byte[]
    transform(  ClassLoader         loader,
                String              className,//className，全类名（包括路径，"/"分割）
                Class<?>            classBeingRedefined,//类定义转换时的Class对象，初始加载时为空
                ProtectionDomain    protectionDomain,//protection...
                byte[]              classfileBuffer)//加载的Class字节码数据
        throws IllegalClassFormatException;
}


```



Demo https://juejin.cn/post/6944881900636864526

# 3 Java Instrument的实现是基于JVM哪种机制？JVMTI是什么，可以做什么？

> 1. 基于JVMTI代理程序；
> 2. JVMTI：一套代理程序机制，为JVM相关工具提供的本地编程接口集合；
> 3. JVMTI可以支持第三方工具程序以代理的方式连接和访问JVM，并利用JVMTI提供的丰富的编程接口，完成很多跟JVM相关的功能；



# 5 Instrument premain、agentmain方法中两个参数agentArgs、inst代表什么？分别会有什么作用？

> 1. agentArgs：代理程序命令行中输入参数，随同“-javaagent”一起传入，与main函数不同的是，这个参数是一个字符串而不是一个字符串数组；
> 2. inst：java.lang.instrument.Instrumentation实例，由JVM自动传入，集中了几乎所有功能方法，如：类操作、classpath操作等；

# 6 java.lang.instrument.ClassFileTransformer是什么，有什么作用？

> 1. ClassFileTransformer当中的transform方法可以对类定义进行操作修改；
> 2. 在类字节码载入JVM前，JVM会调用ClassFileTransformer.transform方法，从而实现对类定义进行操作修改，实现AOP功能；相对于JDK 动态代理、CGLIB等AOP实现技术，不会生成新类，也不需要原类有接口；

> 

# 8 META-INF/MAINFEST.MF参数清单？

> 1. Premain-Class：指定包含premain方法的类名；
> 2. Agent-Class：指定包含agentmain方法的类名；
> 3. Boot-Class-Path：指定引导类加载器搜索的路径列表。查找类的特点于平台的机制失败后，引导类加载器会搜索这些路径；
> 4. Can-Redefine-Class：是否能重新定义此代理所需的类，默认为false；
> 5. Can-Retransform-Class：是否能重新转换此代理所需的类，默认为false；
> 6. Can-Set-Native-Method-Prefix：是否能设置此代理所需的本机方法前缀，默认值为false；

# 9 两个核心API ClassFileTransformer、Instrumention？

> 1. ClassFileTransformer：定义了类加载前的预处理类；
> 2. Instrumentation：增强器
>
> （1）add/removeTransformer：添加/删除ClasFileTransformer；
>
> （2）retransformerClasses：指定哪些类，在已加载的情况下，重新进行转换处理，即触发重新加载类定义；对于重新加载的类不能修改旧有的类声明，比如：不能增加属性、不能修改方法声明等；
>
> （3）redefineClasses：指定哪些类，触发重新加载类定义，与上面不同的是不会重新进行转换处理，而是把处理结果bytecode直接给JVM；
>
> （4）getAllLoadedClasses：获取当前已加载的Class集合；
>
> （5）getInitiatedClasses：获取由某个特定ClassLoader加载的类定义；
>
> （6）getObjectSize：获得一个对象占用的空间大小；
>
> （7）appendToBootstrapClassLoaderSearch/appentToSystemClassLoaderSearch：增加BootstrapClassLoader/SystemClassLoader搜索路径；
>
> （8）isNativeMethodPrefixSupported/SetNativeMethodPrefix：判断JVM是否支持拦截Native Method；

# 10 Java Instrument工作原理？

> 1. 在JVM启动时，通过JVM参数-javaagent，传入agent jar，Instrument Agent被加载；
> 2. 在Instrument Agent 初始化时，注册了JVMTI初始化函数eventHandlerVMinit；
> 3. 在JVM启动时，会调用初始化函数eventHandlerVMinit，启动了Instrument Agent，用sun.instrument.instrumentationImpl类里的方法loadClassAndCallPremain方法去初始化Premain-Class指定类的premain方法；
> 4. 初始化函数eventHandlerVMinit，注册了class解析的ClassFileLoadHook函数；
> 5. 在解析Class之前，JVM调用JVMTI的ClassFileLoadHook函数，钩子函数调用sun.instrument.instrumentationImpl类里的transform方法，通过TransformerManager的transformer方法最终调用我们自定义的Transformer类的transform方法；
> 6. 因为字节码在解析Class之前改的，直接使用修改后的字节码的数据流替代，最后进入Class解析，对整个Class解析无影响；
> 7. 重新加载Class依然重新走5-6步骤；

作者：猿码架构
链接：https://juejin.cn/post/6844903592319516686
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。