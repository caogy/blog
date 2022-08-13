# MyBatis 简介

每个基于 MyBatis 的应用都是一个 SqlSessionFactory 的实例为核心，SqlSessionFactoryBuilder 可以从 XML 配置文件或一个预先配置的 Configuration 对象来构建 SqlSessionFactory 实例。

``` java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

通过源码分析，build 方法将 xml 格式的配置信息转为 Java 的一个对象（Configuration），从而影响 MyBatis 的行为。

![image-20220630114332520](/image/mybatis.webp)

## XML 配置文件

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
    <mappers>
        <!---->
    </mappers>
</configuration>
```

## XML 映射文件

映射文件结构：

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
        
<mapper namespace="">

</mapper>
```

## 核心对象作用域（Scope）和生命周期

### SqlSessionFactoryBuilder

这个类可以被实例化，一旦创建了 SqlSessionFactory ，就不在需要它了，因此 SqlSessionFactoryBuilder 实例的最佳作用域是局部方法变量。

### SqlSessionFactory

SqlSessionFactory 一旦被创建就应该在应用程序运行期一直存在，没有任何理由重新创建另外一个。

### SqlSession

每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的。标准的代码模式：

``` java
try (SqlSession session = sqlSessionFactory.openSession()) {
  // 你的应用逻辑代码
}
```

默认的 openSession() 方法没有参数，它会创建具备如下特性的 SqlSession：

- 事务作用域将会开启（也就是不会自动提交）
- 将由当前环境配置的 DataSource 实例中获取 Connection 对象。
- 事务隔离级别将会使用驱动或数据源的默认设置。
- 预处理语句不会被复用，也不会批量处理更新。

### 映射器实例

映射器最佳作用域为方法变量。

``` java
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  // 你的应用逻辑代码
}
```

## 动态 SQL

### if

``` xml
<if test="true|false">
  string value
</if>
```

### choose

``` xml
<!-- if 标签提供简单的条件判断，无法实现 if ... else if ... else 的逻辑。
要实现上述逻辑需要用到 choose when otherwise 标签：
1、choose 元素包含 when 和 otherwise 两个标签。
2、一个 choose 中至少一个 when。 
3、有 0 个或 1 个 otherwise。
-->

<choose>
  <when test="true|false">
  </when>
    <when test="true|false">
  </when>
  <otherwise>
  </otherwise>
</choose>
```

### where/set/trim

``` xml
<!--
sql where 语句

当 if 条件都不满足时，生成后的 sql 语句不会出现 where 。
如果 if 条件满足，字符串是以  and 开头的，生成 sql 时会自动去除。
这种情况不在需要写 1=1 这样的条件。
-->
<where>
  <if test="true|false">
  and username like concat('%',#{userName},'%')
  </if>
  <if test="true|false">
  and useremail = #{userEmail}
  </if>
</where>

<!--
sql update 语句
-->
update sys_user
<set>
  <if test="true|false">
  
  </if>
  id = #{id}  <!--这条很重要，因为 set 中所有的 if 可能都不满足条件，造成 sql错误-->
</set>
where id = #{id}


<!--
trim 标签有如下属性：
prefix：当 trim 元素内包含内容时，会给内容增加 prefix 指定的前缀。
prefixOverrides：当 trim 元素内包含内容时，会把内容中匹配的前缀字符串去掉。
suffix：当trim 元素内包含内容时，会给内容增加 suffix 指定的后缀。
suffixOverrides：当 trim 元素内包含内容时，会把内容中匹配的后缀字符串去掉。
-->
<
<!--使用 trim 标签实现 where 标签功能如下：-->
<trim prefix="where" prefixOverrides="and |or ">

</trim>

<!--使用 trim 标签实现 set 标签功能如下：-->
<trim prefix="set" suffixOverrides=",">

</trim>
```

### foreach

使用场景：

- sql in 操作。
- 批量插入。
- 动态更新

``` xml
<select id="selectByIdList" resultType="tk.mybatis.simple.model.SysUser">
    select * from sys_user
    where id in
    <foreach collection="idList" open="(" close=")" separator="," item="id" index="i">
        #{id}
    </foreach>
</select>

<!--动态更新，collection="_parameter" 为 MyBatis 内部默认值-->
<update id="updateByMap">
    update sys_user
    set
    <foreach collection="_parameter" item="val" index="key" separator=",">
        ${key}=#{val}
    </foreach>
    where id = #{id}
</update>
```

![foreach](/image/20220813-01-mybatis-foreach.png)
