# **EmbeddedValueResolverAware**

**EmbeddedValueResolverAware 作用**
通过 EmbeddedValueResolverAware 接口可以获取Spring容器加载的 properties文件属性值。

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.context.EmbeddedValueResolverAware;
import org.springframework.stereotype.Component;
import org.springframework.util.StringValueResolver;

/**
 * Created by wangyingjie1 on 2016/9/6.
 */
@Component("propertiesUtils")
public class PropertiesUtils implements EmbeddedValueResolverAware, BeanPostProcessor {

    private StringValueResolver resolver = null;

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.resolver = resolver;
    }

    public String getPropertiesValue(String name) {
        return resolver.resolveStringValue(name);
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {

        System.out.println("yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy");

        return bean;
    }
}
```



```java
import com.google.common.collect.Maps;
import com.jd.caiyu.common.utils.PropertiesUtils;
import com.jd.caiyu.match.domain.agent.enums.AgentTypeEnum;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.Map;

import static com.jd.caiyu.match.domain.agent.enums.AgentTypeEnum.PRIMARY;

/**
 * Created by wangyingjie1 on 2016/9/6.
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("/spring-config.xml")
public class PropertiesUtilsTest {

    @Autowired
    private PropertiesUtils propertiesUtils;

    @Test
    public void testGetValue(){
        String value = propertiesUtils.getPropertiesValue("${jmq.address}");
        Assert.assertEquals("查询结果与期待的不一致！！！", "192.168.x.x:p", value);
        Map<AgentTypeEnum, String> map = Maps.newHashMap();
        map.put(PRIMARY, "xxx");
    }

}

```



## 参考

