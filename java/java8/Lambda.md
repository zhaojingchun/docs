## Lambda



### 方法引用

**总结：**你的引用方法的参数个数、类型，返回值类型要和函数式接口中的方法声明一一对应才行

形式：

- 对象::实例方法
- 类::静态方法
- 类::实例方法

构造器引用：

`ClassName::new`

方法引用的出现，使得我们可以将一个方法赋给一个变量或者作为参数传递给另外一个方法。`::`双冒号作为方法引用的符号，比如下面这两行语句，引用 `Integer`类的 `parseInt`方法。

```java
Function<String, Integer> s = Integer::parseInt;
Integer i = s.apply("10");
```

**Q：什么样的方法可以被引用？**

A：这么说吧，任何你有办法访问到的方法都可以被引用。

**Q：返回值到底是什么类型？**

A：这就问到点儿上了，上面又是 `Function`、又是`Comparator`、又是 `IntBinaryOperator`的，看上去好像没有规律，其实不然。

返回的类型是 Java 8 专门定义的函数式接口，这类接口用 `@FunctionalInterface` 注解。

比如 `Function`这个函数式接口的定义如下：

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

还有很关键的一点，你的引用方法的参数个数、类型，返回值类型要和函数式接口中的方法声明一一对应才行。

比如 `Integer.parseInt`方法定义如下：

```java
public static int parseInt(String s) throws NumberFormatException {
    return parseInt(s,10);
}
```

首先`parseInt`方法的参数个数是 1 个，而 `Function`中的 `apply`方法参数个数也是 1 个，参数个数对应上了，再来，`apply`方法的参数类型和返回类型是泛型类型，所以肯定能和 `parseInt`方法对应上。

这样一来，就可以正确的接收`Integer::parseInt`的方法引用，并可以调用`Funciton`的`apply`方法，这时候，调用到的其实就是对应的 `Integer.parseInt`方法了。

用这套标准套到 `Integer::compare`方法上，就不难理解为什么即可以用 `Comparator<Integer>`接收，又可以用 `IntBinaryOperator`接收了，而且调用它们各自的方法都能正确的返回结果。

`Integer.compare`方法定义如下：

```java
public static int compare(int x, int y) {
    return (x < y) ? -1 : ((x == y) ? 0 : 1);
}
```

返回值类型 `int`，两个参数，并且参数类型都是 `int`。

然后来看`Comparator`和`IntBinaryOperator`它们两个的函数式接口定义和其中对应的方法：

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}

@FunctionalInterface
public interface IntBinaryOperator {
    int applyAsInt(int left, int right);
}
```

对不对，都能正确的匹配上，所以前面示例中用这两个函数式接口都能正常接收。其实不止这两个，只要是在某个函数式接口中声明了这样的方法：两个参数，参数类型是 `int`或者泛型，并且返回值是 `int`或者泛型的，都可以完美接收。

JDK 中定义了很多函数式接口，主要在 `java.util.function`包下，还有 `java.util.Comparator` 专门用作定制比较器。另外，前面说的 `Runnable`也是一个函数式接口。

