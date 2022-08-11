# Spring Boot 多数据源配置

通过注解，实现 master 、slave 数据源动态切换。

## 继承 AbstractRoutingDataSource 类

核心在于：通过继承 `AbstractRoutingDataSource` 抽象类，并重写 `determineCurrentLookupKey` 与 `determineTargetDataSource` 方法。

``` java
public class RoutingDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        String key = RoutingDataSourceContext.getDataSourceRoutingKey();
        return key;
    }

    @Override
    protected DataSource determineTargetDataSource() {
        DruidDataSource ds = (DruidDataSource)super.determineTargetDataSource();
        System.out.println(ds.getRawJdbcUrl());
        return ds;
    }
}
```

在 `determineCurrentLookupKey` 方法中，`RoutingDataSourceContext` 类维护一个 `ThreadLocal<String>` 静态变量。`ThreadLocal` 为线程变量，可以维护一个线程的中间数据。

``` java
public class RoutingDataSourceContext implements AutoCloseable {

    private static final String MASTER_DATASOURCE = "master";

    private static final ThreadLocal<String> threadLocalDataSourceKey = new ThreadLocal<>();

    public static String getDataSourceRoutingKey() {
        String key = threadLocalDataSourceKey.get();
        return key == null ? MASTER_DATASOURCE : key;
    }

    public static void setDataSourceRoutingKey(String key) {
        threadLocalDataSourceKey.set(key);
    }

    public RoutingDataSourceContext(String key) {
        threadLocalDataSourceKey.set(key);
    }

    @Override
    public void close() throws Exception {
        threadLocalDataSourceKey.remove();
    }
}
```

## 注入 bean

注入 master、slave、DataSource、JdbcTemplate 以及 DataSourceTransactionManager。

``` java
@Configuration
public class DataSourceConfiguration {

    @Bean("masterDataSourceProperties")
    @ConfigurationProperties("datasource.master")
    DataSourceProperties masterDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean("slaveDataSourceProperties")
    @ConfigurationProperties("datasource.slave")
    DataSourceProperties slaveDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean
    @Primary
    DataSource dataSource(@Autowired @Qualifier("masterDataSourceProperties") DataSourceProperties master,
                          @Autowired @Qualifier("slaveDataSourceProperties") DataSourceProperties slave) {
        DataSource masterDataSource = master.initializeDataSourceBuilder().build();
        DataSource slaveDataSource = slave.initializeDataSourceBuilder().build();

        RoutingDataSource dataSource = new RoutingDataSource();

        Map<Object, Object> map = new HashMap<>();
        map.put("master", masterDataSource);
        map.put("slave", slaveDataSource);

        dataSource.setTargetDataSources(map);
        dataSource.setDefaultTargetDataSource(masterDataSource);

        return dataSource;
    }

    @Bean
    @Primary
    JdbcTemplate jdbcTemplate(@Autowired DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean
    @Primary
    DataSourceTransactionManager dataSourceTransactionManager(@Autowired DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

## 配置文件与 pom.xml

配置文件 application.yml 需要配置两个数据源，如下：

``` yml
datasource:
  master:
    url: jdbc:mysql://localhost:3380/mk_master?serverTimeZone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
  slave:
    url: jdbc:mysql://localhost:3380/mk_slave?serverTimeZone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
```

pom.xml 依赖如下：

``` pom
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.24</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.7</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```

## 新增注解与 Aspect

方法调用时动态切换数据源，增加如下注解与切片。

``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface DbSetting {
    /**
     * 注解
     *
     * @return
     */
    public String value() default "master";
}

@Component
@Aspect
public class DbSettingAspect {
    @Around("@annotation(dbSetting)")
    public Object dataSourceChange(ProceedingJoinPoint joinPoint, DbSetting dbSetting) throws Throwable {
        try(RoutingDataSourceContext dataSourceContext = new RoutingDataSourceContext(dbSetting.value())){
            return joinPoint.proceed();
        }
    }

}
```

## 动态切换数据源示例

在方法上使用 `DbSetting` 注解。

``` java
@Service
public class UserService {

    @Autowired
    JdbcTemplate jdbcTemplate;

    @DbSetting(value = "master")
    public void showMasterTime() {
        var map = jdbcTemplate.queryForMap("SELECT NOW() as now");
        System.out.println(map);
    }

    @DbSetting(value = "slave")
    public void showSlaveTime() {
        var map = jdbcTemplate.queryForMap("SELECT NOW() as now");
        System.out.println(map);
    }

    public void showDefaultTime(){
        var map = jdbcTemplate.queryForMap("SELECT NOW() as now");
        System.out.println(map);
    }

    public void showExtTime(){
        var map = jdbcTemplate.queryForMap("SELECT NOW() as now");
        System.out.println(map);
    }
}
```

运行 test 查看输入。

``` java
@SpringBootTest
class DynamicDatasourceApplicationTests {

    @Autowired
    UserService userService;

    @Test
    public void testMultiDataSource() {
        System.out.println("showMasterTime：");
        userService.showMasterTime();

        System.out.println("showSlaveTime：");
        userService.showSlaveTime();

        System.out.println("showDefaultTime：");
        userService.showDefaultTime();

        //使用方法内手动切换数据源
        System.out.println("showExtTime：");
        RoutingDataSourceContext.setDataSourceRoutingKey("slave");
        userService.showExtTime();
    }
}
```

输入日志

``` log
showMasterTime：
jdbc:mysql://localhost:3380/mk_master?serverTimeZone=UTC
2022-08-01 15:37:18.574  INFO 15968 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} inited
{now=2022-08-01T15:37:20}
showSlaveTime：
jdbc:mysql://localhost:3380/mk_slave?serverTimeZone=UTC
2022-08-01 15:37:20.146  INFO 15968 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-2} inited
{now=2022-08-01T15:37:20}
showDefaultTime：
jdbc:mysql://localhost:3380/mk_master?serverTimeZone=UTC
{now=2022-08-01T15:37:20}
showExtTime：
jdbc:mysql://localhost:3380/mk_slave?serverTimeZone=UTC
{now=2022-08-01T15:37:20}
```

## 集成 Mybatis

详见仓库 [dynamic-datasource](https://github.com/caogy/dynamic-datasource)。
