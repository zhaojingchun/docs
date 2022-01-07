



```shell
-javaagent:${HOME}/sandbox/lib/sandbox-agent.jar=server.port=10088;server.ip=0.0.0.0 -Dapp.name=solution -Dapp.env=day
```



#### 视频资料

[作者介绍此项目](https://www.bilibili.com/video/BV1Zv411x7mJ?from=search&seid=13810143581544805158&spm_id_from=333.337.0.0)

### 定义

JVM-Sandbox-Repeater是一个基于JVM-Sandbox的服务端录制/回放通用解决方案



怎么调试

https://github.com/oldmanpushcart/greys-anatomy/issues/96



控制台：http://127.0.0.1:8001/online/search.htm









配置类 com.alibaba.jvm.sandbox.repeater.plugin.domain.RepeaterConfig





https://testerhome.com/topics/20868



1. 【本地试用强烈推荐】idea 右键运行启动：jvm-sandbox-repeater/repeater-console/repeater-console-start/src/main/java/com/alibaba/repeater/console/start/Application.java

```shell
jps -l
~/sandbox/bin/sandbox.sh -p 30388 -P 8820
~/sandbox/bin/sandbox.sh  -p  30388 -S
  
# 启动命令
~/sandbox/bin/sandbox.sh -p ${被录制应用进程号} -P ${repeater启动端口}
# 关闭命令
~/sandbox/bin/sandbox.sh -S ${被录制应用进程号}
```





```shell
 java -javaagent:${HOME}/sandbox/lib/sandbox-agent.jar=server.port=8820\;server.ip=0.0.0.0 \
     -Dapp.name=repeater \
     -Dapp.env=daily \
     -jar repeater-console.jar
```

sandBox 事件

https://github.com/alibaba/jvm-sandbox/wiki/EVENT-INTRODUCE



原理篇

https://testerhome.com/topics/21165

尝鲜使用

https://testerhome.com/topics/19778