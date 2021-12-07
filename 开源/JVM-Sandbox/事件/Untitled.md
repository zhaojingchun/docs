

如果model里有 ModuleEventWatcher对象 会注入 DefaultModuleEventWatcher

构建事件对象 SpringInstantiateAdvice.watcher(this.eventWatcher).watch();

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



https://www.cnblogs.com/davidwang456/articles/10950595.html