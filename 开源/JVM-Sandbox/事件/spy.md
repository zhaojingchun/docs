

### SPY 初始化

com.alibaba.jvm.sandbox.core.server.jetty.JettyCoreServer#bind

```java
jvmSandbox = new JvmSandbox(cfg, inst);
```

最终执行 com.alibaba.jvm.sandbox.core.util.SpyUtils#init  在 namespaceSpyHandlerMap里放了 namespace,new EventListenerHandler 对象



加载 module 模块

com.alibaba.jvm.sandbox.core.manager.impl.DefaultCoreModuleManager#reset



开始加载jar 文件  ModuleJarLoader#load()