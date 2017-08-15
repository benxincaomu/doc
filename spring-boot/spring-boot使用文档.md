# spring-boot 简易使用文档

官方文档http://docs.spring.io/spring-boot/docs/1.5.3.RELEASE/reference/htmlsingle/#using-boot-starter

官方文档很详细，也比较容易读懂，想深入了解的可以阅读官方文档，本文档只作为入门使用。

## 创建基础项目

在IDEA中，创建一个spring-boot项目很简单。

`File` ->`New` -> `project`

在弹出的窗口中，左侧选择 `Spring Initializr`,面板右侧选择jdk版本（1.7及以上），`initializr url` 保持默认即可，`Next`。

项目的基础配置，`Type` 选择 `Maven Project` , 其他项根据需要填入即可，`Next`。

选择需要的依赖，通常来说，做web开发需要选择 `Web -> Web`,数据存储选择 `Sql - > Mysql`,orm选择 `Sql -> Jpa` 或者 `Sql -> Mybatis`,为了演示方便，这里选用jpa,  `Next` 。

输入项目名称和路径，`Packaging` 选择 `jar`,完成项目的创建。查看生产的pom.xml，如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>

```

查看生成的`DemoApplication.java`

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}

```

此时运行`DemoApplication.main()`,会发现启动失败，原因是没有数据库相关的配置，接下来进行相关配置。

## 配置文件

spring boot 可以使用 properties文件或者yml文件进行配置。这里选择使用yml文件，先将将`application.properties`重命名为 `application.yml`。

### 数据源

创建demo库，并在application.yml加入以下配置。

```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/demo
    username: root
    password: sftadmin
```

spring boot 默认使用的数据源是`org.apache.tomcat.jdbc.pool.DataSource`，如需使用其他数据源，添加maven依赖后在`datasource`配置中添加`type`来指定要使用的数据源，或者使用代码进行相关配置。

重新运行`DemoApplication.main()` , 出现以下日志，即启动正常。

```verilog
2017-05-15 11:03:11.507  INFO 15928 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-05-15 11:03:11.510  INFO 15928 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 3.792 seconds (JVM running for 4.221)
```

访问http://127.0.0.1:8080,出现了一个错误页面，代码为404,是因为项目里并没有配置默认页面。

### 默认访问页面

在`resources/static/`下创建index.html,刷新页面，错误页面消失，展现出index.html

### Jpa配置

在application.yml中的`spring`标签下添加下面的配置

```yaml
spring: # 只需要在上面的标签下添加即可，不需要重复定义spring
  jpa:
    hibernate: # spring jpa是基于hibernate实现的，所以jpa的配置主要就是hibernate的配置
      ddl-auto: update # 启动时根据代码中的实体类创建数据表，如有变化则更新
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    show-sql: true # 进行jdbc操作时输出sql语句
```

## 开始进行开发

这里使用简单的CRUD操作来进行web和数据库的相关操作。

### Entity

```java
package com.example.demo.web.user;

import org.hibernate.annotations.GenericGenerator;

import javax.persistence.*;
import java.util.Date;


@Table(name="user" ,indexes = {@Index(columnList = "user_name",name="un_idx",unique = true)})
public class User {
    @Id
    @GenericGenerator(name = "uuid",strategy = "uuid")
    @GeneratedValue(generator = "uuid")
    @Column(name = "id",length = 36)
    private String id;
    @Column(name = "user_name",length = 20)
    private String userName;
    @Column(name = "password")
    private String password;
    @Column(name = "create_date")
    private Date createDate;
}
```

启动项目，发现数据库中多了一张user表。

### UserRepository

```java
package com.example.demo.web.user;


import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.web.PageableDefault;
/**
* 继承PagingAndSortingRepository 之后，接口中会有默认的增删改查以及分页、排序方法
* PagingAndSortingRepository<User,String> User为要操作的实体类，String为对应的主键类型。
* 当使用自增主键时，User类中的主键类型应使用Integer或Long作为属性类型，避免多余的拆箱和装箱
* 
*/
public interface UserRepository extends PagingAndSortingRepository<User,String> { 
    Page<User> findByUserNameLike(String userName,@PageableDefault( value = 10,sort = {"createTime"}, direction = Sort.Direction.DESC) 				Pageable pageable);
}
```

repository是`spring data jpa`的核心。详细使用文档

### UserService

```java
package com.example.demo.web.user;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;

@Service
public class UserService {
    @Resource
    private UserRepository userRepository;

    public User saveUser(User user) {
        return userRepository.save(user);
    }

    public void deleteUser(String id) {
        userRepository.delete(id);
    }

    public User findUser(String id) {
        return userRepository.findOne(id);
    }

    public Page<User> findUsers(String userName, Pageable pageable) {
        return userRepository.findByUserNameLike(userName, pageable);
    }
}
```

### UserController

```java
package com.example.demo.web.user;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;

@RestController //此类内的所有方法不会返回页面
@RequestMapping("user")  //请求路径
public class UserController {
    @Resource 
    private UserService userService;

    @PostMapping("save")  
    public User saveUser(User user){
        return userService.saveUser(user);
    }
    @PostMapping("delete/{id}")  //获取路径中的参数
    public void deleteUser(@PathVariable("id") String id){
        userService.deleteUser(id);
    }
    @GetMapping("findUsers")
    public Page<User> findUsers(String userName, Pageable pageable){
        return  userService.findUsers(userName,pageable);
    }
    @GetMapping("findUser")
    public User findUser(String id){
        return userService.findUser(id);
    }
}
```

## 代码配置

`spring boot` 提供了很多jar，是我们可以通过简单的配置完成以前spring的xml复杂配置，另外，也提供很多注解，方便开发管理第三方类库的bean。

### 配置bean

以DataSource配置为例，先添加Druid数据源依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.28</version>
</dependency>
```

使用代码配置数据源

```java
package com.example.demo.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;
import java.sql.SQLException;

@Configuration
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class}) //exclude 要排除的Configuration
@EnableTransactionManagement
public class DataSourceConfig {
    @Bean(name = "dataSource", initMethod = "init", destroyMethod = "close")
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource() {
        DruidDataSource druidDataSource = new DruidDataSource();
      
        return druidDataSource;
    }

    @Bean
    public FilterRegistrationBean filterRegistrationBean() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new WebStatFilter());
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
        return filterRegistrationBean;
    }

    @Bean
    public ServletRegistrationBean druidServlet() {
        return new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
    }
}
```

### Web应用配置

```java
package com.example.demo.config;


import com.alibaba.fastjson.serializer.PropertyFilter;
import com.alibaba.fastjson.serializer.SerializerFeature;
import com.alibaba.fastjson.support.config.FastJsonConfig;
import com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.HandlerMethodReturnValueHandler;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.config.annotation.*;

import java.util.List;

@Configuration
@EnableAutoConfiguration(exclude = {JacksonAutoConfiguration.class})
public class WebAppConfigurer extends WebMvcConfigurerAdapter {
    /**
     * 配置MessageConverters,这里使用了fastjson,需要先引入依赖
     * @param converters 已配置的消息转换器
     */
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.clear();
        FastJsonConfig fastjsonConfig = new FastJsonConfig();
        fastjsonConfig.setDateFormat("yyyy-MM-dd HH:mm:ss");
        SerializerFeature[] features = { SerializerFeature.WriteMapNullValue, SerializerFeature.WriteDateUseDateFormat,
                SerializerFeature.WriteNullNumberAsZero,SerializerFeature.WriteEnumUsingToString };
        fastjsonConfig.setSerializerFeatures(features);
        PropertyFilter propertyFilter=(Object object, String name, Object value)->value!=null;//如果值为null，则不输出该字段
        fastjsonConfig.setSerializeFilters(propertyFilter);
        FastJsonHttpMessageConverter fsConverter = new FastJsonHttpMessageConverter();
        fsConverter.setFastJsonConfig(fastjsonConfig);
        converters.add(fsConverter);
        super.configureMessageConverters(converters);
    }
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        super.extendMessageConverters(converters);
    }

    /**
     * 增加拦截器
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        super.addInterceptors(registry);
    }


    /**
     * 视图解析器配置，前后端分离时不需要做修改
     * @param registry
     */
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        super.configureViewResolvers(registry);
    }

    /**
     * 参数解析
     * @param argumentResolvers 已配置的参数解析器
     */
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        super.addArgumentResolvers(argumentResolvers);
    }

    /**
     * 返回值拦截器
     * @param returnValueHandlers 已配置的拦截器
     */
    @Override
    public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> returnValueHandlers) {
        super.addReturnValueHandlers(returnValueHandlers);
    }

    /**
     * 异常处理
     * @param exceptionResolvers 已配置的异常处理器
     */
    @Override
    public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> exceptionResolvers) {
        super.extendHandlerExceptionResolvers(exceptionResolvers);
    }
}

```

## starters

#### 官方提供

这里我只列出个人觉得比较常用或者是以前可以解决以前开发中遇到的问题的starter,详细见官方文档

| 名称                               | 描述                                       |
| -------------------------------- | ---------------------------------------- |
| spring-boot-starter-web          | 使用spring MVC构建应用，使用tomcat作为默认容器          |
| spring-boot-starter-mail         | 邮件相关操作                                   |
| spring-boot-starter-data-redis   | redis                                    |
| spring-boot-starter-data-mongodb | mongodb                                  |
| spring-boot-starter-jdbc         | 使用tomcat jdbc连接池进行jdbc操作                 |
| spring-boot-starter-websocket    | websocket(从服务器推送到页面)                     |
| spring-boot-starter-data-jpa     | jpa                                      |
| spring-boot-starter-jetty        | 使用jetty作为servlet容器，如果要使用jetty，需要先将依赖中的tomcat依赖排除 |
| spring-boot-starter-tomcat       | tomcat作为spring-boot的默认容器，由`spring-boot-starter-web`默认加载其依赖 |
| spring-boot-starter-logging      | 使用`logback`作为默认的日志输出组件，由`spring-boot-starter-web`默认加载其依赖 |
| spring-boot-starter-log4j2       | 使用`log4j2`输出日志，需先排除`spring-boot-starter-logging`依赖 |



#### 第三方提供



| 名称                          | 描述        |
| --------------------------- | --------- |
| mybatis-spring-boot-starter | 整合mybatis |



