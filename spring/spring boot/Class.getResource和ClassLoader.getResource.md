## Class.getResource和ClassLoader.getResource

Class.getResource(String path)

```
path不以’/'开头时，默认是从此类所在的包下取资源；
path  以’/'开头时，则是从ClassPath根下获取；
```

Class.getClassLoader().getResource(String path)

```
path不能以’/'开头时；
path是从ClassPath根下获取；
```

