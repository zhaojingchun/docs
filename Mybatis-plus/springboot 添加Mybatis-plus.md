# Mybatis-plus



### 添加maven依赖

```java
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <scope>runtime</scope>
  </dependency>
<dependency>
  <groupId>com.zaxxer</groupId>
  <artifactId>HikariCP</artifactId>
</dependency>
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
</dependency>
```



### 添加plus config

```java
@Configuration
@MapperScan(basePackages = {"com.xxx.dao"}, sqlSessionFactoryRef = "msMasterSqlSessionFactory")
public class MasterDataSourceConfig {

    /**
     *
     */
    @Bean
//    @Primary
    @ConfigurationProperties(prefix = "spring.datasource.msmaster")
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        return dataSource;
    }

    @Bean("msMasterSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource, ResourceLoader resourceLoader) throws Exception {
        MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();
        /* 数据源 */
        sqlSessionFactory.setDataSource(dataSource);
        /* 枚举扫描 */
        sqlSessionFactory.setTypeEnumsPackage("com.xxx.domain");
        /* xml扫描 */
        sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath*:mapper/*.xml"));
        /* 扫描 typeHandler */
//        sqlSessionFactory.setTypeHandlersPackage("com.baomidou.mybatisplus.samples.mysql.type");
        MybatisConfiguration configuration = new MybatisConfiguration();
        configuration.setJdbcTypeForNull(JdbcType.NULL);
        /* 驼峰转下划线 */
        configuration.setMapUnderscoreToCamelCase(true);
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        sqlSessionFactory.setPlugins(mybatisPlusInterceptor);
        /* map 下划线转驼峰 */
        configuration.setObjectWrapperFactory(new MybatisMapWrapperFactory());
        sqlSessionFactory.setConfiguration(configuration);
        /* 自动填充插件 */
//        globalConfig.setMetaObjectHandler(new MysqlMetaObjectHandler());
//        sqlSessionFactory.setGlobalConfig(globalConfig);
        return sqlSessionFactory.getObject();
    }

}

```



### 添加数据库连接池配置

```properties
spring:
  datasource:
    msmaster:
      jdbc-url: jdbc:mysql://xxxxx:3306/repeater?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&autoReconnect=true&useSSL=false
      driver-class-name: com.mysql.jdbc.Driver
      username: root
      password: 
      type: com.zaxxer.hikari.HikariDataSource
      timeBetweenEvictionRunsMillis: 300000
      minEvictableIdleTimeMillis: 3600000
      testWhileIdle: true
      hikari:
        maximum-pool-size: 10
        minimum-idle: 5

```

