

### 核心方法

com.alibaba.jvm.sandbox.core.server.jetty.JettyCoreServer#bind

### 模块加载

com.alibaba.jvm.sandbox.core.manager.impl.DefaultCoreModuleManager#reset

加载模块

com.alibaba.jvm.sandbox.core.manager.impl.ModuleJarLoader#loadingModules

#### 成员变量

com.alibaba.jvm.sandbox.core.manager.impl.DefaultCoreModuleManager#injectResourceOnLoadIfNecessary

加载 RepeaterModule

com.alibaba.jvm.sandbox.repeater.module.RepeaterModule#loadCompleted



发放录制数据 信息

com.alibaba.jvm.sandbox.repeater.plugin.core.impl.api.DefaultInvocationListener#onInvocation

### 初始化 JVM-Sandbox 配置类

加载主类 ： com.alibaba.jvm.sandbox.core.CoreConfigure

```java
;
cfg=/Users/xxx/sandbox/cfg;
system_module=/Users/xxx/sandbox/module;
mode=agent;
sandbox_home=/Users/xxx/sandbox;
user_module=/Users/xxx/sandbox/sandbox-module;
provider=/Users/xxxx/sandbox/provider;
namespace=default;
server.ip=0.0.0.0;
server.port=8820;
```

/Users/xxx/sandbox/cfg/sandbox.properties

调动 toConfigure ()

```properties
#
# this properties file define the sandbox's config
# @author luanjia@taobao.com
# @date   2016-10-24
#

# define the sandbox's ${SYSTEM_MODULE} dir
## system_module=../module

# define the sandbox's ${USER_MODULE} dir, multi values, use ',' split
## user_module=~/.sandbox-module;~/.sandbox-module-1;~/.sandbox-module-2;~/.sandbox-module-n;
user_module=~/.sandbox-module;
#user_module=/home/staragent/plugins/jvm-sandbox-module/sandbox-module;/home/staragent/plugins/monkeyking;

# define the sandbox's ${PROVIDER_LIB} dir
## provider=../provider

# define the network interface
## server.ip=0.0.0.0

# define the network port
## server.port=4769

# define the server http response charset
server.charset=UTF-8

# switch the sandbox can enhance system class
unsafe.enable=true

```



### 模块加载

/Users/xxx/sandbox/module/sandbox-mgr-module.jar

/Users/x/.sandbox-module/repeater-bootstrap.jar  == repeater-console.jar

/Users/xxxx/.sandbox-module/repeater-module.jar





com.alibaba.jvm.sandbox.core.server.ProxyCoreServer





java.util.ServiceLoader#load(java.lang.Class<S>, java.lang.ClassLoader)？？？？