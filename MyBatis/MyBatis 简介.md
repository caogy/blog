每个基于 MyBatis 的应用都是一个 SqlSessionFactory 的实例为核心，SqlSessionFactoryBuilder 可以从 XML 配置文件或一个预先配置的 Configuration 对象来构建 SqlSessionFactory 实例。

``` java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

通过源码分析，build 方法将 xml 格式的配置信息转为 Java 的一个对象（Configuration），从而影响 MyBatis 的行为。

![image-20220630114332520](/image/mybatis.webp)

### XML 配置文件

配置文件结构如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
        
<configuration>
    <properties/>
    <settings/>
    <typeAliases/>
    <typeHandlers/>
    <objectFactory />
    <plugins />
    <environments>
        <environment>
            <transactionManager />
            <dataSource />
        </environment>
    </environments>
    <databaseIdProvider />
    <mappers />
</configuration>
```

