

## mock 测试

[TOC]

[参考](https://juejin.cn/post/6844903924248346637)

[文本教程](https://www.kancloud.cn/apachecn/howtodoinjava-zh/1953197)

### 简介

我们在做开发的过程中，难免会有写外部依赖没法提供，一般遇到这种情况我们是要进行mock测试的。如果不需要对静态方法，私有方法等特殊进行验证测试，则仅仅使用 Spring boot 自带的 **Mockito** 即可完成相关的测试数据 Mock。若需要则可以使用 PowerMock，简单实用，结合 Spring 可以使用注解注入。

Mockito 更多的使用可查看→[官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fstatic.javadoc.io%2Forg.mockito%2Fmockito-core%2F2.8.47%2Forg%2Fmockito%2FMockito.html%2312)

### 直接上代码

**@MockBean**

**SpringBoot 在执行单元测试时，会将该注解的 Bean 替换掉 IOC 容器中原生 Bean。**

例如下面代码中， ProjectService 中通过 ProjectMapper 的 selectById 方法进行数据库查询操作：

```java
@Service
public class ProjectService {

    @Autowired
    private ProjectMapper mapper;
    public ProjectDO detail(String id) {
        return mapper.selectById(id);
    }
}
```

此时我们可以对 Mock 一个 ProjectMapper 对象替换掉 IOC 容器中原生的 Bean，来模拟数据库查询操作，如：

```java 
@RunWith(SpringRunner.class)
@SpringBootTest
public class ProjectServiceTest {
  
    @MockBean
    private ProjectMapper mapper;
    @Autowired
    private ProjectService service;

    @Test
    public void detail() {
        ProjectDemoDO model = new ProjectDemoDO();
        model.setId("1");
        model.setName("dubbo-demo");
        Mockito.when(mapper.selectById("1")).thenReturn(model);
        ProjectDemoDO entity = service.detail("1");
        assertThat(entity.getName(), containsString("dubbo-demo"));
    }

}

```

