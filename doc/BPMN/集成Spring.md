# 集成Spring

### `ProcessEngineFactoryBean`

通过配置 `org.flowable.spring.ProcessEngineFactoryBean` 来处理流程引擎配置和创建引擎。

下面是一个示例：

```xml
<bean id="processEngineConfiguration" class="org.flowable.spring.SpringProcessEngineConfiguration">
    <property name="jdbcUrl" value="jdbc:h2:mem:flowable;DB_CLOSE_DELAY=1000" />
    <property name="jdbcDriver" value="org.h2.Driver" />
    <property name="jdbcUsername" value="sa" />
    <property name="jdbcPassword" value="" />
    <property name="databaseSchemaUpdate" value="true" />
</bean>

<bean id="processEngine" class="org.flowable.spring.ProcessEngineFactoryBean">
    <property name="processEngineConfiguration" ref="processEngineConfiguration" />
</bean>
```

注意这里的 `ProcessEngineConfiguration` 使用的是 `org.flowable.spring.SpringProcessEngineConfiguration`

### 事务

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <bean id="processEngineConfiguration" class="org.flowable.spring.SpringProcessEngineConfiguration">
        <property name="dataSource" ref="dataSource" />
        <property name="transactionManager" ref="transactionManager" />
        <property name="databaseSchemaUpdate" value="true" />
    </bean>

    <bean id="processEngine" class="org.flowable.spring.ProcessEngineFactoryBean">
        <property name="processEngineConfiguration" ref="processEngineConfiguration" />
    </bean>

    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        <property name="driverClass" value="org.h2.Driver" />
        <property name="url" value="jdbc:h2:mem:flowable;DB_CLOSE_DELAY=1000" />
        <property name="username" value="sa" />
        <property name="password" value="" />
    </bean>

    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <bean id="repositoryService" factory-bean="processEngine" factory-method="getRepositoryService" />

    <bean id="runtimeService" factory-bean="processEngine" factory-method="getRuntimeService" />

    <bean id="taskService" factory-bean="processEngine" factory-method="getTaskService" />

    <bean id="historyService" factory-bean="processEngine" factory-method="getHistoryService" />

    <bean id="managementService" factory-bean="processEngine" factory-method="getManagementService" />

    <tx:annotation-driven />

</beans>
```

上面的xml代码中，当给 `SpringProcessEngineConfiguration` 设置数据源时， `SpringProcessEngineConfiguration` 内部将会使用 `org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy` 代理数据源，这是为了保证从数据源获取的SQL连接与Spring的事务可以协同工作。

![Snipaste_2021-09-30_17-39-22.png](../../img/BPMN/Snipaste_2021-09-30_17-39-22.png)

如果自行在Spring配置中声明了 `TransactionAwareDataSourceProxy` ，最好不要将它用在已经配置Spring事务的资源上（例如 `DataSourceTransactionManager` 与 `JPATransactionManager` ）。即使你配置了，这些事务资源还是使用的未代理数据源。

![Snipaste_2021-09-30_17-45-29.png](../../img/BPMN/Snipaste_2021-09-30_17-45-29.png)

`<tx:annotation-driven />` 是用于开启事务注解

你可以使用下面的代码创建由上面的配置文件配置的Spring应用上下文：

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("flowable.cfg.xml");
```

或者如果你想要在单元测试中使用Spring应用上下文你可以：

```java
@FlowableTest
@ContextConfiguration("classpath:flowable.cfg.xml")
public class TransactionTest {



}
```

