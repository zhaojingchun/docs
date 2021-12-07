### spring-boot + swagger

[TOC]

[swagger 接入 springboot](https://juejin.cn/post/6844904004741234696)

[注解说明](https://blog.csdn.net/thinkwon/article/details/107477801)

### 1、Swagger 介绍

很多人都以为 `Swagger` 只是一个接口文档生成框架，其实并不是。 Swagger  是一个围绕着 `OpenAPI Specification`（OAS，中文也称 OpenAPI规范）构建的一组开源工具。可以帮助你从 API 的设计到 API 文档的输出再到  API 的测试，直至最后的 API 部署等整个 API 的开发周期提供相应的解决方案，是一个庞大的项目。 Swagger  不仅免费，而且开源，不管你是企业用户还是个人玩家，都可以使用 Swagger 提供的方案构建令人惊艳的 `REST API`。

Swagger 有几个主要的产品。

- [Swagger Editor](https://link.juejin.cn?target=http%3A%2F%2Feditor.swagger.io%2F%3F_ga%3D2.112541447.2078165713.1574600445-3923049.1574128700) – 一个基于浏览器的 Open API 规范编辑器。
- [Swagger UI](https://link.juejin.cn?target=https%3A%2F%2Fswagger.io%2Fswagger-ui%2F) – 一个将 OpenAPI 规范呈现为可交互在线文档的工具。
- [Swagger Codegen](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fswagger-api%2Fswagger-codegen) – 一个根据 OpenAPI 生成调用代码的工具。

如果你想了解更多信息，可以访问 Swagger 官方网站 [swagger.io](https://link.juejin.cn?target=https%3A%2F%2Fswagger.io)。

### 2、Springfox 介绍

源于 Java 中 Spring 框架的流行，让一个叫做  Marrty Pitt 的老外有了为 SpringMVC 添加接口描述的想法，因此他创建了一个遵守 OpenAPI 规范（OAS）的项目，取名为 *swagger-springmvc*，这个项目可以让 Spring 项目自动生成 JSON 格式的 OpenAPI 文档。这个框架也仿照了 Spring 项目的开发习惯，使用注解来进行信息配置。

后来这个项目发展成为 `Springfox`，再后来扩展出 `springfox-swagger2` ，为了让 JSON 格式的 API 文档更好的呈现，又出现了 `springfox-swagger-ui` 用来展示和测试生成的 OpenAPI 。这里的 springfox-swagger-ui 其实就是上面介绍的 Swagger-ui，只是它被通过 webjar 的方式打包到 jar 包内，并通过 maven 的方式引入进来。

上面提到了 Springfox-swagger2 也是通过注解进行信息配置的，那么是怎么使用的呢？下面列举常用的一些注解，这些注解在下面的 Springboot 整合 Swagger 中会用到。

| **注解**          | **示例**                                                     | **描述**                                   |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------ |
| @ApiModel         | @ApiModel(value = "用户对象")                                | 描述一个实体对象                           |
| @ApiModelProperty | @ApiModelProperty(value = "用户ID", required = true, example = "1000") | 描述属性信息，执行描述，是否必须，给出示例 |
| @Api              | @Api(value = "用户操作 API(v1)", tags = "用户操作接口")      | 用在接口类上，为接口类添加描述             |
| @ApiOperation     | @ApiOperation(value = "新增用户")                            | 描述类的一个方法或者说一个接口             |
| @ApiParam         | @ApiParam(value = "用户名", required = true)                 | 描述单个参数                               |

更多的 Springfox 介绍，可以访问 Springfox 官方网站。

[Springfox Reference Documentation (http://springfox.github.io)](https://link.juejin.cn/?target=http%3A%2F%2Fspringfox.github.io%2Fspringfox%2Fdocs%2Fcurrent%2F)

### 3、Springboot 整合 Swagger

3.1、引入maven

```xml
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.9.0</version>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.9.0</version>
</dependency>
新的ui页面 [swagger ui 访问地址](http://localhost:8080/doc.html)
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>swagger-bootstrap-ui</artifactId>
    <version>1.9.6</version>
</dependency>
```

3.2、配置 Springfox-swagger

Springfox-swagger 的配置通过一个 Docket 来包装，Docket 里的 apiInfo 方法可以传入关于接口总体的描述信息。而 apis 方法可以指定要扫描的包的具体路径。在类上添加 @Configuration 声明这是一个配置类，最后使用 @EnableSwagger2 开启 Springfox-swagger2。@ConditionalOnProperty(name = "swagger.enable", havingValue = "true") 通过@ConditionalOnProperty可以决定不同环境是否提供swagger服务。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * <p>
 * Springfox-swagger2 配置
 *
 * @Author niujinpeng
 * @Date 2019/11/19 23:17
 */
@Configuration
@EnableSwagger2
@ConditionalOnProperty(name = "swagger.enable", havingValue = "true")
public class SwaggerConfig {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("net.codingme.boot.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("未读代码 API")
                .description("公众号：未读代码(weidudaima) springboot-swagger2 在线借口文档")
                .termsOfServiceUrl("https://www.codingme.net")
                .contact("达西呀")
                .version("1.0")
                .build();
    }
}
```

#### 3.3 properties 配置

```properties
swagger:
  enable: true
```

#### 3.4. 代码编写

文章不会把所有代码一一列出来，这没有太大意义，所以只贴出主要代码，完整代码会上传到 Github，并在文章底部附上 Github 链接。

参数实体类 `User.java`，使用 `@ApiModel` 和 ` @ApiModelProperty` 描述参数对象，使用 ` @NotNull` 进行数据校验，使用 `@Data` 为参数实体类自动生成 get/set 方法。

```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.format.annotation.DateTimeFormat;

import javax.validation.constraints.NotNull;
import java.util.Date;

/**
 * <p>
 * 用户实体类
 *
 * @Author niujinpeng
 * @Date 2018/12/19 17:13
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@ApiModel(value = "用户对象")
public class User {

    /**
     * 用户ID
     *
     * @Id 主键
     * @GeneratedValue 自增主键
     */
    @NotNull(message = "用户 ID 不能为空")
    @ApiModelProperty(value = "用户ID", required = true, example = "1000")
    private Integer id;

    /**
     * 用户名
     */
    @NotNull(message = "用户名不能为空")
    @ApiModelProperty(value = "用户名", required = true)
    private String username;
    /**
     * 生日
     */
    @DateTimeFormat(pattern = "yyyy-MM-dd hh:mm:ss")
    @ApiModelProperty(value = "用户生日")
    private Date birthday;
}

```

编写 Controller 层，使用 `@Api` 描述接口类，使用 `@ApiOperation` 描述接口，使用 `@ApiParam` 描述接口参数。代码中在查询用户信息的两个接口上都添加了 ` tags = "用户查询"` 标记，这样这两个方法在生成 Swagger 接口文档时候会分到一个共同的标签组里。

```java
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import lombok.extern.slf4j.Slf4j;
import net.codingme.boot.domain.Response;
import net.codingme.boot.domain.User;
import net.codingme.boot.enums.ResponseEnum;
import net.codingme.boot.utils.ResponseUtill;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import javax.validation.constraints.NotNull;
import java.util.ArrayList;
import java.util.List;

/**
 * <p>
 * 用户操作
 *
 * @Author niujinpeng
 * @Date 2019/11/19 23:17
 */

@Slf4j
@RestController(value = "/v1")
@Api(value = "用户操作 API(v1)", tags = "用户操作接口")
public class UserController {

    @ApiOperation(value = "新增用户")
    @PostMapping(value = "/user")
    public Response create(@Valid User user, BindingResult bindingResult) throws Exception {
        if (bindingResult.hasErrors()) {
            String message = bindingResult.getFieldError().getDefaultMessage();
            log.info(message);
            return ResponseUtill.error(ResponseEnum.ERROR.getCode(), message);
        } else {
            // 新增用户信息 do something
            return ResponseUtill.success("用户[" + user.getUsername() + "]信息已新增");
        }
    }

    @ApiOperation(value = "删除用户")
    @DeleteMapping(value = "/user/{username}")
    public Response delete(@PathVariable("username")
                           @ApiParam(value = "用户名", required = true) String name) throws Exception {
        // 删除用户信息 do something
        return ResponseUtill.success("用户[" + name + "]信息已删除");
    }

    @ApiOperation(value = "修改用户")
    @PutMapping(value = "/user")
    public Response update(@Valid User user, BindingResult bindingResult) throws Exception {
        if (bindingResult.hasErrors()) {
            String message = bindingResult.getFieldError().getDefaultMessage();
            log.info(message);
            return ResponseUtill.error(ResponseEnum.ERROR.getCode(), message);
        } else {
            String username = user.getUsername();
            return ResponseUtill.success("用户[" + username + "]信息已修改");
        }
    }

}

```

### 4、注解说明

#### 4.1、@Api

用在请求的类上，表示对类的说明

| **注解属性** | **类型** | **描述**                                |
| ------------ | -------- | --------------------------------------- |
| **tags**     | String[] | 描述请求类的作用，非空时会覆盖value的值 |
| **value**    | String   | 描述请求类的作用                        |

示例

```java
@Api(tags = {"用户controller"})
public class UserController {}
```

#### 4.2、@ApiOperation

用在请求类的方法上，说明方法的用途和作用

| **注解属性** | **类型** | **描述**       |
| ------------ | -------- | -------------- |
| **value**    | String   | 方法的简要说明 |
| **notes**    | String   | 方法的备注说明 |

示例

```java
@GetMapping("/list")
@ApiOperation(value = "查询用户列表")
public List<UserDTO> list() {
    return list;
}
```

#### 4.3、@ApiParam

可用在方法，参数和字段上，一般用在请求体参数上，描述请求体信息

| **注解属性**     | **类型** | **描述**                                                     |
| ---------------- | -------- | ------------------------------------------------------------ |
| **name**         | String   | 参数名称，参数名称可以覆盖方法参数名称，路径参数必须与方法参数一致 |
| **value**        | String   | 参数的简要说明                                               |
| **required**     | boolean  | 参数是否必须传，默认为 false （路径参数必填）                |
| **defaultValue** | String   | 参数的默认值                                                 |

示例

```java
@PostMapping
@ApiOperation(value = "新增用户")
public Boolean insert(@RequestBody @ApiParam(name = "UserDTO", value = "新增用户参数") UserDTO userDTO) {
    list.add(userDTO);
    return true;
}
```

#### 4.4、@ApiImplicitParams

用在请求的方法上，表示一组参数说明，里面是`@ApiImplicitParam`列表

#### 4.5、@ApiImplicitParam

用在 `@ApiImplicitParams` 注解中，一个请求参数的说明

| **注解属性**     | **类型** | **描述**                                                     |
| ---------------- | -------- | ------------------------------------------------------------ |
| **name**         | String   | 参数名称，参数名称可以覆盖方法参数名称，路径参数必须与方法参数一致 |
| **value**        | String   | 参数的说明、解释                                             |
| **required**     | boolean  | 参数是否必须传，默认为 false （路径参数必填）                |
| **paramType**    | String   | 参数的位置，header 请求参数的获取：`@RequestHeader`；query 请求参数的获取：`@RequestParam`；path（用于 restful 接口）–> 请求参数的获取：`@PathVariable`；body（不常用）；form（不常用） |
| **dataType**     | String   | 参数类型，默认 String，其它值 dataType=“Integer”             |
| **defaultValue** | String   | 参数的默认值                                                 |

示例

```java
@GetMapping("/page")
@ApiOperation(value = "分页查询问题列表")
@ApiImplicitParams({
    @ApiImplicitParam(name = "pageNum", value = "当前页数"),
    @ApiImplicitParam(name = "pageSize", value = "每页记录数")
})
public List<UserDTO> page(
    @RequestParam(defaultValue = "1", required = false) Integer pageNum, @RequestParam(defaultValue = "10", required = false) Integer pageSize) {
    return list;
}
```

#### 4.6、@ApiResponses

用在请求的方法上，表示一组响应

#### 4.7、@ApiResponse

用在 `@ApiResponses` 中，一般用于表达一个错误的响应信息

| **注解属性** | **类型** | **描述**                    |
| ------------ | -------- | --------------------------- |
| **code**     | int      | 响应状态码                  |
| **message**  | String   | 信息，例如 “请求参数没填好” |
| **response** | Class<?> | 抛出异常的类                |

示例

```java
@PutMapping
@ApiOperation(value = "更新用户信息")
@ApiResponses({
    @ApiResponse(code = 400, message = "请求参数没填好"),
    @ApiResponse(code = 404, message = "请求路径没有或页面跳转路径不对")
})
public Boolean update(@RequestBody @ApiParam(name = "UserDTO", value = "更新用户参数") UserDTO userDTO) {}

```

#### 4.8、@ApiModel

用在实体类（模型）上，表示相关实体的描述。

| **注解属性**    | **类型** | **描述**       |
| --------------- | -------- | -------------- |
| **value**       | String   | 模型的备用名称 |
| **description** | String   | 该类的详细说明 |

示例

```java
@ApiModel(value = "用户", description = "查询用户")
public class UserDTO implements Serializable
```

#### 4.9、@ApiModelProperty

用在实体类属性上，表示属性的相关描述。

| **注解属性** | **类型** | **描述**                   |
| ------------ | -------- | -------------------------- |
| **value**    | String   | 属性简要描述               |
| **name**     | String   | 重写属性名称               |
| **dataType** | Stirng   | 重写属性类型               |
| **required** | boolean  | 参数是否必传，默认为 false |
| **example**  | Stirng   | 属性示例                   |

示例

```java
@ApiModelProperty(value = "用户id")
private Integer userId;

@ApiModelProperty(value = "用户名")
private String username;
```



### Q&A

#### 1、返回值带泛型

**第一步：返回Result加上泛型注解：Result<Order>**

第二步：@ApiOperation(value="根据ID查找",notes="根据ID查询RTU设置数据",response = Result.class) 不能有response属性

泛型的属性值得有get set 方法

```java
@ApiOperation(value = "获取镜像打包结果")
@PostMapping("/fetchPullImageResult")
@ResponseBody
@ApiResponses(@ApiResponse(code = 60001, message = "无权限"))
public ResultModel<PullImageResult> fetchPullImageResult(@RequestBody @Validated  Param param){
  //校验token
  ResultModel<PullImageResult> result = ResultModel.instance();
  return result;
}
```



#### 2、访问页面

```
http://solution-api.jd.com:8080/swagger-ui.html#/
http://solution-api.jd.com:8080/doc.html
```

