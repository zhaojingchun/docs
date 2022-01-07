

#### 

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava-parent</artifactId>
    <version>19.0</version>
</dependency>
```

//new 一个 array List 数组

```java
Lists.newArrayList(em)
```

//一维转二维

```
Lists.partition
```

##### 获取host

```
InetAddress.getLocalHost().getHostAddress()
```



```
System.getProperty(key)
```





apache.commons.lang3

```
MethodUtils
```



只执行一次

```
private static AtomicBoolean initFlag = new AtomicBoolean();
initFlag.compareAndSet(false, true)
```