# EventWatchBuilder 事件观察者类构建器



构建代码

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

