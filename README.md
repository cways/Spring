# Spring

## 1 目录结构

1. entity包,放置项目中实体类(一个表一个类),pojo
2. util工具包,各种工具类(StringHelper类)
3. dao接口包,各种操作接口类(ICRM_UserDao)
4. dao.impl实现接口包,各种实现操作接口的实现类(CRM_UserDaoImpl)
5. service接口包,业务实现的接口(ICRM_UserService)
6. service.impl实现业务接口实现类(CRM_UserServiceImpl)
7. controller包,控制器实现类(CRM_UserController)
8. config(不是文件夹,是个资源包),放置SpringMVC和Hibernate的配置文件

## 2 架构项目包之间的引用关系

请求--->controller--->service--->dao--->db

## 3 整合框架

### 3.1 下载SpringMVC & Hibernate框架
### 3.2 导入SSH框架的jar包

1. 导入SpringMVC的JAR包
    aop
    beans
    context
    context-support
    core
    expressions
    test
    tx
    web
    webmvc
    ...

2. 导入Hibernate的Jar包
    lib/required目录下全部

3. 导入三方依赖
    日志包:common-logging

4. db驱动包
    mysql:mysql-connectioin

## 4 配置web.xml

### 4.1 配置Spring的IOC容器


```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext-*.xml</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

### 4.2 配置SpringMVC的DispatcherServlet

```
<servlet>
    <servlet-name>springDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>WEB-INF/dispatcher-servlet.xml</param-value>
    </init-param>
</servlet>
<servlet-mapping>
    <servlet-name>springDispatcherServlet</servlet-name>
    <url-pattern>url</url-pattern>
</servlet-mapping>
```

### 4.3 springmvc编码方式过滤器

```
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 4.4 为了使用SpringMVC框架实现RUSTful风格,请求配置HiddenHttpMethodFilter

```
<filter>
    <filter-name>hidderHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>hidderHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 5 配置SpringMVC

主要负责加载标注@Controller类、打开注解的处理器适配器、注解的处理器映射器、视图解析器等等

### 5.1 导入xml的命名空间

    beans
    context
    mvc

### 5.2 配置自动扫描的包

```
<context:component-scan base-package="com.cways.controller.*"></context:component-scan>
```
### 5.3 配置注解的映射器和配置器以及其他配置

```
<mvc:annotation-driven></mvc:annotation-driven>
```

### 5.4 处理静态资源问题

```
<mvc:default-servlet-handler/>
```

### 5.3 配置视图解析器

```
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/"></property>
    <property name="suffix" value=".*"></property>
</bean>
```

## 6 配置Spring

### 6.1 配置命名空间

同上

### 6.2 配置自动扫描的包
除了以下类型的包,全都交给Spring管理.

```
<context:component-scan base-package="com.cways" use-default-filters="false">
    <context:exclude-filter type="annotation" expression = "org.springframework.stereotype.Controller"/>
    <context:exclude-filter type="annotation" expression = "org.springframework.web.bind.annotation.ControllerAdvice" />
</context:component-scan>
```

## 7 applicationContext-dao.xml配置

applicationContext-dao.xml文件中主要负责配置：加载db.properties、配置数据源、配置SqlSessionFactoryBean、Mapper扫描器

### 7.1 加载db.properties文件内容

```
<context:property-placeholder location = "/WEB-INF/db.properties"/>
```

### 7.2 配置数据源

1. 使用的dbcp连接池（common-dbcp）

```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="maxActive" value="30"/>
    <property name="maxIdle" value="5"/>
</bean>
```

2. 使用DruidDataSource连接池（druid）

```
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="maxActive" value="30"/>
    <property name="maxIdle" value="5"/>
</bean>
```

### 7.3 配置SqlSessionFactory

1. 使用Mybatis（org.mybatis.spring.boot:mybatis-spring-boot-starter）

```
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 数据源 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 加载mybatis的全局配置文件 -->
        <property name="configLocation" value="classpath:mybatis/sqlMapConfig.xml"/>
    </bean>
```

2. 使用hibernate

### 7.4 配置Mapper扫描器

```
<!-- 配置 Mapper扫描器 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!--扫描包路径，如果需要扫描多个包中间使用半角逗号隔开-->
    <property name="basePackage" value="com.cways.mapper"/>
    <!--这里不适用ref='sqlSessionFactory'是因为上面加载配置文件导致这边引用会报错-->
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
```

## 8 配置applicationContext-service.xml

    负责扫描业务层组件

### 8.1 扫描标注@Repository注解的service

```
<context:component-scan base-package="com.cways.service.impl.*"/>
```

## 9 配置applicationContext-transaction.xml

    负责处理事务等（这个需要了解spring aop、代理模式、反射等）

### 9.1 事务管理器 对mybatis操作数据库事务控制，spring使用jdbc的事务控制类

```
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!--数据源dataSource在applicationContext-dao.xml中配置了-->
    <property name="dataSource" ref="dataSource"/>
</bean>
```

### 9.2 通知

```
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="save*" propagation="REQUIRED"/>
        <tx:method name="delete*" propagation="REQUIRED"/>
        <tx:method name="insert*" propagation="REQUIRED"/>
        <tx:method name="update*" propagation="REQUIRED"/>
        <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
        <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
        <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
    </tx:attributes>
</tx:advice>
```

### 9.3 aop

```
<aop:config>
    <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.cways.service.impl.*.*(..))"/>
</aop:config>
```