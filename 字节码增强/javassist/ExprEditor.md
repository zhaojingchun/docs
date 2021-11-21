

## ExprEditor

### 总结



### 说明

方法主体的翻译。
用户可以定义该类的子类来自定义如何修改方法体。整体架构类似于策略模式。

如果在CtMethod中调用instrument()，则从头到尾扫描方法体。每当找到一个表达式，比如一个方法调用和一个新表达式(对象创建)，就会在expreit中调用edit()。Edit()可以检查和修改给定的表达式。修改反映在原方法体上。如果edit()什么也不做，原始的方法主体就不会改变。

下面的代码是一个示例:

```java
 CtMethod cm = ...;
 cm.instrument(new ExprEditor() {
     public void edit(MethodCall m) throws CannotCompileException {
         if (m.getClassName().equals("Point")) {
             System.out.println(m.getMethodName() + " line: "
                                + m.getLineNumber());
     }
 });
```

这段代码检查cm表示的方法中出现的所有方法调用，并打印类Point中声明的方法的名称和行号。此代码不修改cm表示的方法体。如果必须修改方法体，调用MethodCall中的replace()。

| Modifier and Type | Method                                  | Description                                           |
| :---------------- | :-------------------------------------- | :---------------------------------------------------- |
| `boolean`         | `doit(CtClass clazz, MethodInfo minfo)` | Undocumented method.                                  |
| `void`            | `edit(Cast c)`                          | 编辑用于显式类型转换的表达式(可重写)。                |
| `void`            | `edit(ConstructorCall c)`               | 编辑构造函数调用(可重写)。                            |
| `void`            | `edit(FieldAccess f)`                   | 编辑字段访问表达式(可重写)。                          |
| `void`            | `edit(Handler h)`                       | Edits a catch clause (overridable).                   |
| `void`            | `edit(Instanceof i)`                    | Edits an instanceof expression (overridable).         |
| `void`            | `edit(MethodCall m)`                    | 编辑方法调用(可重写)。                                |
| `void`            | `edit(NewArray a)`                      | Edits an expression for array creation (overridable). |
| `void`            | `edit(NewExpr e)`                       | Edits a `new` expression (overridable).               |



## MethodCall

| Modifier and Type  | Method                                | Description                                                  |
| :----------------- | :------------------------------------ | :----------------------------------------------------------- |
| `java.lang.String` | `getClassName()`                      | 返回目标对象的类名，该方法是在该对象上调用的。               |
| `java.lang.String` | `getFileName()`                       | Returns the source file containing the method call.          |
| `int`              | `getLineNumber()`                     | Returns the line number of the source line containing the method call. |
| `CtMethod`         | `getMethod()`                         | Returns the called method.                                   |
| `java.lang.String` | `getMethodName()`                     | Returns the name of the called method.                       |
| `java.lang.String` | `getSignature()`                      | 返回方法签名 (the parameter types and the return type).      |
| `boolean`          | `isSuper()`                           | Returns true if the called method is of a superclass of the current class. |
| `CtClass[]`        | `mayThrow()`                          | Returns the list of exceptions that the expression may throw. |
| `void`             | `replace(java.lang.String statement)` | Replaces the method call with the bytecode derived from the given source text. |
| `CtBehavior`       | `where()`                             | Returns the method or constructor containing the method-call expression represented by this object. |

