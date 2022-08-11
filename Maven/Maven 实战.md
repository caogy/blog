# Maven 实战

使用 Idea 新建一个 Maven 项目，结构如下：

``` tree
│─pom.xml
└─src
    ├─main
    │  ├─java
    │  └─resources
    └─test
        └─java
```

pom.xml 文件默认内容。

``` pom
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.tyy</groupId>
    <artifactId>mvndemo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

</project>
```

## Maven 常用的命令

- 编译：`mvn clean compile`
- 测试：`mvn clean test`
- 打包：`mvn clean package`
- 本地安装：`mvn clean install`
- 依赖树：`mvn dependency:tree`
- 依赖列表：`mvn dependency:list`

执行顺序：compile->test->package-install

## 坐标和依赖

Maven 坐标：groupId、artifactId、version 是需要定义的，packaging 是可选的（默认 jar），而 classfier 是不能直接定义的。

### 依赖范围

Maven 依赖范围有：compile、test、provided、runtime、system。缺省时默认：compile。

依赖范围(Scope)|编译|测试|运行时|例子
---|---|---|---|---
compile|需要|需要|需要|spring-core
test|-|需要|-|JUnit
provided|需要|需要|-|servlet-api 由容器提供，比如Tomcat
runtime|-|需要|需要|JDBC 驱动实现
system|需要|需要|-|本地的，Maven 仓库之外的类库文件

## 生命周期和插件

Maven 的生命周期是为了对所有构建过程进行抽象与统一。这个生命周期包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有构建步骤，Maven 的生命周期是抽象的，这意味着生命周期本身不做任何实际的工作。这些的实际工作由插件来完成。每个构建步骤都可以绑定一个或多个插件行为，而且 Maven 为大多数构建步骤编写并绑定了默认插件，如：编译的插件`maven-compiler-plugin`，测试的插件`maven-surefire-plugin`等。

Maven 拥有三套相互独立的生命周期，分别为：clean、default、site。clean生命周期的目的是清理项目，default生命周期的目的是构建项目，而site生命周期的目的是建立项目站点。

每个生命周期有包含多个阶段（phase）。

## 聚合与继承

### 聚合

聚合可以将多个模块同时编译、打包。需要修改`pom.xml`文件，增加`modules`配置节点，并将`packaging` 改为`pom`。

``` xml
<packaging>pom</packaging>

<modules>
    <module>module1</module>
    <module>module2</module>
    <module>web</module>
</modules>
```

代码层级结构如下：

![多模块](/image/20220811-01-multi-module.png)

此时将`java-multi-module`模块称为聚合模块，在聚合模块执行`mvn`命令时，会先解析聚合模块的pom，分析要构建的模块、并计算出一个反应堆构建顺序(Reactor Build Order)，然后根据这个顺序一次构建各个模块。

### 继承

需要将父模块(`java-multi-module`)的`packaging`改为`pom`，在子模块(`module1`)的`pom`文件增加`parent`配置。

可以被继承的pom元素，经常使用到的：

- groupId：项目组ID，项目坐标的核心元素。
- version：项目版本，项目坐标的核心元素。
- description：项目描述。
- properties：自定义的Maven属性。
- dependencies：项目的依赖配置。
- dependencyManagement：项目的依赖管理配置
- repositories：项目的仓库属性。
- build：包括项目的源码目录配置、输出目录配置、插件配置、插件管理配置等。
- reporting：包括项目的输出目录配置、报告插件配置等。

### 聚合与继承的关系

可以将聚合与继承在同一个模块配置。

### 约定优于配置

Maven 会假设用户的项目是这样的：

- 源码目录为 src/main/java/
- 资源文件目录为 src/main/resources/
- 单元测试源码目录为：src/test/java/
- 编译输出目录为 target/classes/
- 打包方式为 jar
- 包输出目录为 target/

这些默认配置在超级pom文件，该文件在：`$MAVEN_HOME/lib/maven-model-builder-x.x.x.jar中的org/apache/maven/model/pom-4.0.0.xml`

## Maven 配置

### 1、Idea 中 Maven 配置

修改 Maven home directory、User setting file、Local repository，目录路径按照本机 Maven 安装目录，如下示例：

![maven](/image/20220807-01-maven.png)

### 2、配置 settings.xml 中心仓库

``` xml
<repositories>
   <repository>
      <id>public</id>
      <name>aliyun nexus</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <releases>
         <enabled>true</enabled>
      </releases>
   </repository>
</repositories>
<pluginRepositories>
   <pluginRepository>
      <id>public</id>
      <name>aliyun nexus</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <releases>
         <enabled>true</enabled>
      </releases>
      <snapshots>
         <enabled>false</enabled>
      </snapshots>
   </pluginRepository>
</pluginRepositories>
```

### 3、默认打包 `jar` 不能运行

在 `mvn clean package` 默认打包生成的`jar`是不能直接运行的，因为带有 `main`方法的类信息不会添加到 `manifest`中（解压`jar`文件后的 `META-INF/MANIFEST.MF` 文件中无法看到 `Main-Class`一行记录），需要在pom.xml 文件增加以下配置：

``` pom
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.2.2</version>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>PackageName.MainClassName</mainClass>
                    </manifest>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 4、使用MyBatis时xml文件复制

``` xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
        </resource>
    </resources>
</build>
```

### 5、打包时禁用 test

``` xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <skipTests>true</skipTests>
            </configuration>
        </plugin>
    </plugins>
</build>
```
