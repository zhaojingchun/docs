# ClassPool 类池

ClassPool对象是CtClass对象的容器。一旦创建了一个CtClass对象，它将永远记录在一个ClassPool中。这是因为编译器稍后在编译引用由CtClass表示的类的源代码时可能需要访问CtClass对象。

例如，假设一个新的方法getter()被添加到一个表示Point类的CtClass对象中。稍后，程序尝试编译源代码，包括对Point中的getter()的方法调用，并使用编译后的代码作为方法的主体，该方法将被添加到另一个类Line。如果表示Point的CtClass对象丢失，编译器就无法编译对getter()的方法调用。注意，原始的类定义不包括getter()。因此，要正确地编译这样的方法调用，ClassPool必须在程序执行期间包含CtClass的所有实例。

## 避免内存不足

如果对象存在惊人大量的```CtClass```，```ClassPool```的这种规范则可能会引起极大的内存消耗（这其实很少会发生，因为javassist会以各种方式降低内存开销：冻结calss等方式）。为了避免这种问题，你需要从```ClassPool```中明确移除不必要的```CtClass```对象。

可以调用```CtClass```的```detach()```方法，然后会将该对象从```ClassPool```中移除：
```java
ClassPool classPool = ClassPool.getDefault();
CtClass cc = classPool.get("org.byron4j.cookbook.javaagent.Javassist2ClassPool");

// 调用该方法后，会将CtClass对象从ClassPool中移除
cc.detach();
```

调用```CtClass.detach()```方法后，你不应该再继续调用这个ctClass对象的其他方法了。可是，你可以调用```ClassPool.get()```方法会重新读取class文件构造一个表示相同的类的ctClass对象。


另一方案是,偶尔使用一个新的classPool代替旧的classPool。如果旧的classPool被gc了，包含在其中的ctClass也会被gc掉。
```java
ClassPool cp = new ClassPool(true)
```

## 级联ClassPool

***如果程序运行在一个web应用中***,是有必要创建```ClassPool```的多实例的。每一个类加载器应该只创建一个ClassPool对象。此时程序不应该使用```getDefault()```方法而是使用ClassPool的构造方法。

多个ClassPool对象可以像```java.lang.ClassLoader```一样级联：

```java
// 级联ClassPool
ClassPool parent = ClassPool.getDefault();
ClassPool child = new ClassPool(parent);
child.insertClassPath("./classes");
```

如果调用了```child.get()```方法，child首先委托父ClassPool，如果父ClassPool加载class文件失败，然后child再尝试从```./classes/```目录下查找类文件。

如果```child.childFirstLookup```设置为true，则child尝试在委托给父classPool之前去加载class文件：
```java
// child classpool在委托之前加载类文件
ClassPool parent2 = ClassPool.getDefault();
ClassPool child2 = new ClassPool(parent2);
child2.appendSystemPath();         // 和默认同样的class查找路径
child2.childFirstLookup = true;    // 改变child的行为
```

## 更改类名以定义新类

一个新的class可以被定义为一个已存在的类的副本。
```java
// 首先获取表示Point类的一个CtClass对象
// 然后调用setName修改类名
ClassPool pool3 = ClassPool.getDefault();
CtClass cc3 = pool3.get("org.byron4j.cookbook.javaagent.Point");
cc3.setName("Pair");
```

这个程序首先包含类Point的ctClass对象，然后调用```setName()```方法为CtClass对象设置新的名称。在这个调用之后，所有由该CtClass对象定义的所有类名展示均由Point变更为Pair，类定义的其他部分则不会变更。

注意： ```CtClass.setName()```方法改变了ClassPool中记录的对象。从实现的角度来看，一个ClassPool对象是CtClass对象的哈希表形式。```setName```会改变CtClass对象在哈希表中关联的key，key值从原来的类名变更为新的设置后的类名。
因此，如果后面再调用```get("Point")```方法的话，不会再返回变量cc3引用的CtClass对象了，ClassPool对象会再次读取class文件Point.class并且为Point类构造一个新的CtClass对象，这是因为ClassPool对象中关联Point的key不存在了：
```java
ClassPool pool = ClassPool.getDefault();
CtClass ctClass = pool.get("org.byron4j.cookbook.javaagent.Point");
CtClass ctClass1 = pool.get("org.byron4j.cookbook.javaagent.Point");
ctClass.setName("Pair");
CtClass ctClass2 = pool.get("Pair");
CtClass ctClass3 = pool.get("org.byron4j.cookbook.javaagent.Point");
System.out.println(ctClass == ctClass2);  // true;
System.out.println(ctClass3 == ctClass2); // false;
```

>```ClassPool``` 对象用于维护类和CtClass的一对一映射关系。javassist不允许两个不一样的CtClass表示同一个class，除非是两个独立的ClassPool创建的。


为了创建一个默认ClassPool实例(Clas.getDefault()返回的)的一个副本，可以使用以下代码片段：
```java
ClassPool cp = new ClassPool(true);
```
这样一来，你拥有了两个ClassPool对象，可以从每一个ClassPool提供不同的CtClass对象表示同一个类。

```java
ClassPool pool10 = ClassPool.getDefault();
CtClass ctClass10 = pool10.get("org.byron4j.cookbook.javaagent.Point");
ClassPool pool20 = new ClassPool(true);
CtClass ctClass20 = pool20.get("org.byron4j.cookbook.javaagent.Point");
System.out.println(pool10 == pool20);   // false 不同的ClassPool中表示同一个类的CtClass对象
```

## 通过重命名一个冻结的CtClass来创建一个新的CtClass对象

一旦一个CtClass对象已经被writeFile()或者toBytecode()方法转到class文件，Javassist拒绝进一步修改该CtClass对象。因此，如果代表Point类的CtClass对象冻结后不能通过setName()修改它的名称。
```java
ClassPool pool30 = ClassPool.getDefault();
CtClass ctClass30 = pool30.get("org.byron4j.cookbook.javaagent.Point");
ctClass30.writeFile();// 被冻结
ctClass30.setName("Pair");// 错误
```

为了打破这个约束，可以使用ClassPool的```getAndRename()```方法：
```java
ClassPool pool30 = ClassPool.getDefault();
CtClass ctClass30 = pool30.get("org.byron4j.cookbook.javaagent.Point");
ctClass30.writeFile();// 被冻结
//ctClass30.setName("Pair");// 冻结后不能使用--错误
pool30.getAndRename("org.byron4j.cookbook.javaagent.Point", "Pair");
```

因为如果getAndRename方法被调用了，ClassPool首先读取Point.class文件来创建一个新的CtClass对象，会在记录到ClassPool的哈希表之前将Point变更为Pair。
getAndRename方法的源码：
```java
public CtClass getAndRename(String orgName, String newName)
        throws NotFoundException
{
    // 获取一个新的CtClass对象
    CtClass clazz = get0(orgName, false);
    if (clazz == null)
        throw new NotFoundException(orgName);

    if (clazz instanceof CtClassType)
        ((CtClassType)clazz).setClassPool(this);

    // 设置新的名称
    clazz.setName(newName);         // indirectly calls
                                    // classNameChanged() in this class
    return clazz;
}
```
