



### 保存数据入库 

com.alibaba.repeater.console.service.impl.RecordServiceImpl#saveRecord

```sql
CREATE TABLE `record` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `gmt_create` datetime NOT NULL COMMENT '创建时间',
  `gmt_record` datetime NOT NULL COMMENT '录制时间',
  `app_name` varchar(255) NOT NULL COMMENT '应用名',
  `environment` varchar(255) NOT NULL COMMENT '环境信息',
  `host` varchar(36) NOT NULL COMMENT '机器IP',
  `trace_id` varchar(32) NOT NULL COMMENT '链路追踪ID',
  `entrance_desc` varchar(2000) NOT NULL COMMENT '链路追踪ID',
  `wrapper_record` longtext NOT NULL COMMENT '记录序列化信息',  //body
  `request` longtext NOT NULL COMMENT '请求参数JSON',
  `response` longtext NOT NULL COMMENT '返回值JSON',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8 COMMENT='录制信息';
```





### 保存本地文件 

com.alibaba.jvm.sandbox.repeater.plugin.core.impl.standalone.StandaloneBroadcaster

```
RecordModel
```

### 保存数据库

com.alibaba.jvm.sandbox.repeater.plugin.core.impl.standalone.StandaloneBroadcaster



recordWrapper 和 recordModel 之间的关系  **等同**

```java
   public RecordWrapper(RecordModel recordModel) {
        this.timestamp = recordModel.getTimestamp();
        this.appName = recordModel.getAppName();
        this.environment = recordModel.getEnvironment();
        this.host = recordModel.getHost();
        this.traceId = recordModel.getTraceId();
        if (recordModel.getEntranceInvocation() instanceof HttpInvocation) {
            this.entranceDesc = ((HttpInvocation)recordModel.getEntranceInvocation()).getRequestURL();
        } else {
            this.entranceDesc = recordModel.getEntranceInvocation().getIdentity().getUri();
        }
        this.entranceInvocation = recordModel.getEntranceInvocation();
        this.subInvocations = recordModel.getSubInvocations();
    }
```

### RecordWrapper

```java
private long timestamp;

    private String appName;

    private String environment;

    private String host;

    private String traceId;
    /**
     * 入口描述
     */
    private String entranceDesc;
    /**
     * 入口调用
     */
    private Invocation entranceInvocation;
    /**
     * 子调用信息
     */
    private List<Invocation> subInvocations;
```

####  RecordModel

```java
    private long timestamp;

    private String appName;

    private String environment;

    private String host;

    private String traceId;

    private Invocation entranceInvocation;

    private List<Invocation> subInvocations;
```



### 文件存

文件直接存的   RecordModel





## 查询详情



反序列化有此方法：

com.alibaba.repeater.console.service.convert.RecordDetailConverter#**convert**



### 如参出参 序列化 

com.alibaba.jvm.sandbox.repeater.plugin.core.wrapper.SerializerWrapper#inTimeSerialize



DefaultInvocationListener

### traceId 生成

```
TraceGenerator.generate();
```

