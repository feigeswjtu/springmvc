Web如果没有数据库，就像电脑没有磁盘一样，无法存储或者存储的量特别少，所以SpringMVC继承Database是最重要的功能之一，这一章我们讲解一下Spring怎么继承Mysql。
项目名称是**database**，可以到我[github](https://github.com/feigeswjtu/springmvc-projects)下载。

# 数据库连接池Druid
生产环境中肯定需要数据库连接池，我们使用aliyun提供的Druid数据库连接池，官方文档是这样介绍的: Druid是Java语言中最好的数据库连接池。Druid能够提供强大的监控和扩展功能。
Druid的官方文档见: [Druid](https://github.com/alibaba/druid)

# 持久层框架Mybatis

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

我们使用Mybatis来演示持久层的功能。
# 引入必要的包
从本章开始，引入的包会越来越多，为了更好的管理包版本，我们定义一些properties
```xml
<properties>
    <java.version>1.8</java.version>
    <spring.version>4.3.12.RELEASE</spring.version>
</properties>
```

引入包:
```xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.yaml</groupId>
            <artifactId>snakeyaml</artifactId>
            <version>1.17</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.1</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.1</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.6</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>6.0.6</version>
        </dependency>
```
这些包功能如下:
1. org.springframework >> spring-tx: 提供transaction的功能，是SpringMVC链接数据库必备的。
2. org.springframework >> spring-jdbc: 提供各种数据库的jdbc的功能，是SpringMVC链接数据库必备的。
3. org.yaml >> snakeyaml: 项目的一些常量配置信息使用yml进行配置，这个包用来映射yml文件到properties中，yml是比xml、properties、json文件都要简单的一种文件格式，[wikipedia](https://en.wikipedia.org/wiki/YAML)
4. mybatis: mybatis官方包。
5. mybatis-spring: spring对mybatis支持的包，[官网](http://www.mybatis.org/spring/zh/index.html)。
6. druid: alibaba提供的一个数据库连接池的包。
7. mysql-connector-java: mysql的链接包。

# 配置代码
## 配置YAML的FactoryBean和Property配置管理器PropertySourcesPlaceholderConfigurer
在RootConfig
```java
    @Bean
    public static YamlPropertiesFactoryBean yamlPropertiesFactoryBean() {
        YamlPropertiesFactoryBean propertiesFactoryBean = new YamlPropertiesFactoryBean();
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        try {
            // 加载classpath*:config/下的所有yml文件作为配置源
            Resource[] resources = resolver.getResources("classpath*:config/*.yml");
            propertiesFactoryBean.setResources(resources);
        } catch (IOException e) {
            throw new IllegalStateException(e);
        }
        return propertiesFactoryBean;
    }

    @Bean
    public static PropertySourcesPlaceholderConfigurer configurer() {
        PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
        //yamlPropertiesFactoryBean的配置信息作为配置
        configurer.setProperties(yamlPropertiesFactoryBean().getObject());
        return configurer;
    }
```
## Druid相关配置
首先在config下建立一个properties包，包含一些配置Class，新建一个类: DruidConfig。

```java
@Component
public class DruidConfig {
    @Value("${druid.url}")
    private String url;
    @Value("${druid.username}")
    private String username;
    @Value("${druid.password}")
    private String password;
    @Value("${druid.initialSize}")
    private Integer initialSize;
    @Value("${druid.minIdle}")
    private Integer minIdle;
    @Value("${druid.maxActive}")
    private Integer maxActive;
    @Value("${druid.maxWait}")
    private Long maxWait;
    @Value("${druid.timeBetweenEvictionRunsMillis}")
    private Long timeBetweenEvictionRunsMillis;
    @Value("${druid.minEvictableIdleTimeMillis}")
    private Long minEvictableIdleTimeMillis;
    @Value("${druid.validationQuery}")
    private String validationQuery;
    @Value("${druid.testWhileIdle}")
    private Boolean testWhileIdle;
    @Value("${druid.testOnBorrow}")
    private Boolean testOnBorrow;
    @Value("${druid.testOnReturn}")
    private Boolean testOnReturn;
    //省略set和get方法
}
```
建立一个配置文件config/database.yml，内容如下:
```yml
druid:
  url: jdbc:mysql://192.168.2.107:3306/mall
  username: root
  password: feige
  initialSize: 10
  minIdle: 5
  maxActive: 100
  maxWait: 60000
  timeBetweenEvictionRunsMillis: 60000
  minEvictableIdleTimeMillis: 300000
  validationQuery: SELECT '1'
  testWhileIdle: true
  testOnBorrow: false
  testOnReturn: false
```
Druid数据库连接池所需要的配置信息就有了，后面我们专门讲一下Druid的其他用法，我们这里只是使用它比较简单的功能。


## Database相关配置
Database是比较独立的配置，所以我们单独建立一个JavaConfig来编写它的一些配置，Class名称是com.cyf.base.config.DatabaseConfig。
```java
@Configuration
public class DatabaseConfig {

    @Bean(initMethod = "init", destroyMethod = "close")
    public DruidDataSource myDataSource(DruidConfig druidConfig) {
        DruidDataSource ds = new DruidDataSource();
        //使用com.mysql.cj.jdbc.Driver作为连接渠驱动
        ds.setDriverClassName("com.mysql.cj.jdbc.Driver");
        configDataSource(druidConfig, ds);
        return ds;
    }

    static void configDataSource(DruidConfig druidConfig, DruidDataSource ds) {
        ds.setUrl(druidConfig.getUrl());
        ds.setUsername(druidConfig.getUsername());
        ds.setPassword(druidConfig.getPassword());
        ds.setInitialSize(druidConfig.getInitialSize());
        ds.setMinIdle(druidConfig.getMinIdle());
        ds.setMaxActive(druidConfig.getMaxActive());
        ds.setMaxWait(druidConfig.getMaxWait());
        ds.setTimeBetweenEvictionRunsMillis(druidConfig.getTimeBetweenEvictionRunsMillis());
        ds.setMinEvictableIdleTimeMillis(druidConfig.getMinEvictableIdleTimeMillis());
        ds.setValidationQuery(druidConfig.getValidationQuery());
        ds.setTestWhileIdle(druidConfig.getTestWhileIdle());
        ds.setTestOnBorrow(druidConfig.getTestOnBorrow());
        ds.setTestOnReturn(druidConfig.getTestOnReturn());
    }

    @Bean
    public SqlSessionFactoryBean mySqlSession(@Qualifier("myDataSource") DataSource dataSource) throws IOException {
        String path = "classpath*:mapper/*xml";
        // classpath*:mapper/下的所有xml配置都生成SqlBean
        return getSqlSession(dataSource, path);
    }

    // Mybatis的MapperScanner配置
    @Bean
    public MapperScannerConfigurer myMapperScanner() {
        MapperScannerConfigurer configurer = new MapperScannerConfigurer();
        configurer.setBasePackage("com.cyf.base.persist.mybatis.dao");
        configurer.setSqlSessionFactoryBeanName("mySqlSession");
        return configurer;
    }

    // SqlSessionFactoryBean，数据库连接必备
    static SqlSessionFactoryBean getSqlSession(DataSource dataSource, String path) throws IOException {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        Resource[] resources = resolver.getResources(path);
        sqlSessionFactoryBean.setMapperLocations(resources);
        return sqlSessionFactoryBean;
    }
}
```

#测试
## Dao接口

新建接口com.cyf.base.persist.mybatis.dao.TestDao
```java
public interface TestDao {
    Integer selectOneNumber();
}
```

## 建立Mapper文件
Dao层有了，需要一个Mapper对应真正的sql语句。
resources下建立一个mapper文件； mapper/TestDao.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.cyf.base.persist.mybatis.dao.TestDao">

    <select id="selectOneNumber" resultType="java.lang.Integer">
        SELECT 100;
    </select>

</mapper>
```

## 测试代码
```java
@RestController
@RequestMapping("/api")
public class TestController {
    @Autowired
    private TestDao testDao;

    @GetMapping("/hello")
    public Object hello(String name){
        return testDao.selectOneNumber();
    }
}
```
浏览器中访问:http://localhost:8080/api/hello
返回: 100

到此为止，一个麻雀虽小，五脏俱全的Spring+Mybatis访问Mysql的项目已经完成，希望对大家有帮助。
# 其他
## Mybatis Generator
MyBatis是一个很强大的轻量级框架，为了减少我们自己写sql的麻烦，它提供了一个可以根据我们的表结构，生成持久层需要的Model、Mapper等文件的插件[Mybatis Generator](http://www.mybatis.org/generator/index.html)，只是生成的功能是一些简单的sql，多表join等这些复杂的功能还是需要我们自己写sql。
