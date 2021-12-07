

# RepeaterModule





##### onLoad  加载

```java
public synchronized void watch() {
    new EventWatchBuilder(watcher)
        .onClass("org.springframework.beans.factory.support.SimpleInstantiationStrategy")
        .onBehavior("instantiate")
        .onWatch(new AdviceAdapterListener(new AdviceListener() {
            @Override
            protected void afterReturning(Advice advice) throws Throwable {
                try {
                    /* (RootBeanDefinition beanDefinition, String beanName...) */
                    String beanName = advice.getParameterArray()[1].toString();
                    Object target = advice.getReturnObj();
                    SpringContextInnerContainer.addBean(beanName, advice.getReturnObj());
                    log.info("Register bean:name={},instance={}", beanName, target);
                } catch (Exception e) {
                    log.error("[Error-2000]-register spring bean occurred error.", e);
                }
            }
        }), Type.BEFORE, Type.RETURN);
}
```

watcher 是 DefaultModuleEventWatcher 类的new 对象

字节码增强 com.alibaba.jvm.sandbox.core.manager.impl.DefaultModuleEventWatcher#watch(com.alibaba.jvm.sandbox.core.util.matcher.Matcher, com.alibaba.jvm.sandbox.api.listener.EventListener, com.alibaba.jvm.sandbox.api.resource.ModuleEventWatcher.Progress, com.alibaba.jvm.sandbox.api.event.Event.Type...)