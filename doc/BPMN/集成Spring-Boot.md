# 集成Spring Boot

### 兼容性

flowable的starter支持Spring Boot 2.0+ 和1.5。但它主要支持Spring Boot 2.0+，这就意味着 actuator endpoints 也只支持Spring Boot 2.0+。flowable的starter已经引用了Spring Boot Starter，如何你需要使用其它版本的Spring Boot Starter，你需要自己额外定义。

### 开始

在Spring Boot中使用flowable，你需要导入 `flowable-spring-boot-starter` 或者 `flowable-spring-boot-starter-rest` 的依赖：

```xml
<dependency>
    <groupId>org.flowable</groupId>
    <artifactId>flowable-spring-boot-starter</artifactId>
    <version>${flowable-version}</version>
</dependency>
```

```xml
<dependency>
    <groupId>org.flowable</groupId>
    <artifactId>flowable-spring-boot-starter-rest<artifactId>
    <version>${flowable-version}</version>
</dependency>
```

如果你不需要引入所有的引擎，你可以使用flowable其它的starter，这些将在后面讲到。

// TODO: 补充starter选择

