



http://127.0.0.1:8001/replay/execute.json

```java
ip: 10.0.76.17
mock: true
traceId: 010000076017163790626877610003ed
appName: solution
```

```
# 示例回放地址（工程使用需要维护repeater插件的ip:port替换，指定ip发起回放）
repeat.repeat.url=http://%s:%s/sandbox/default/module/http/repeater/repeat
```





com.alibaba.jvm.sandbox.repeater.plugin.core.impl.spi.RepeatSubscribeSupporter





发送 回放事件 com.alibaba.jvm.sandbox.repeater.module.RepeaterModule#repeat



http://localhost:8080/matrix/list.html





com.alibaba.repeater.console.start.controller.page.RecordController#recordLocal



```
http://localhost:8001/record/list.json?appName
```

存文件方法 ： com.alibaba.jvm.sandbox.repeater.plugin.core.impl.standalone.StandaloneBroadcaster#broadcastRecord

查询记录内容 ： com.alibaba.repeater.console.service.impl.ReplayServiceImpl#fetchRecordByTraceId





 http://localhost:8001/record/fetchRecordByTraceId.json?appName=data-migration&traceId=192168056001163815362108910005ed



反序列化

com.alibaba.repeater.console.service.impl.RecordServiceImpl#getPages