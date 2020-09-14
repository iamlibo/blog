---
title: Spring Boot 笔记
date: 2018-01-26 12:17:13
tags: Spring Boot
---

# Spring Boot 笔记
最近有几个小项目需要进行相关技术预研，使用了Spring Boot这套框架。
整个过程有一些零散，从头入门的教程网上很多了，就不再重复写了。只是把自己认为有重要的或者有坑的地方记录下来。
<!--more-->

## 引用Spring Boot 版本及基础配置

这里使用Spring IO Platform 项目，看介绍是可以减轻包管理版本的难度，直接按照官网加入进来没感觉什么不适。这种方式所有引入的依赖都不需要声明版本。按照这个配置引入Spring Boot 的版本是1.5.9.RELEASE。

pom.xml代码如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.liveup</groupId>
    <artifactId>platform-pom</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.target}</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.spring.platform</groupId>
                <artifactId>platform-bom</artifactId>
                <version>Brussels-SR6</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>
</project>
```
其他项目直接将这个项目做parent就可以了。

```
    <parent>
        <groupId>com.jc.platform</groupId>
        <artifactId>platform</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

```

## Spring Boot 启动类的位置
在开发过程发现在Spring Boot 的启动类不能放根下，就是必须得有一个包。要不然启动时会报错。因为Spring Boot 的自动配置，会从启动类所在包向下进行查找配置。类似于@ComponentScan的配置。

## Spring Boot 启动方式
Spring Boot 正常启动（方式之一）是自己写一个启动类，加上@SpringBootApplication标签。
代码如下：
```
@SpringBootApplication
public class PlatformApplication {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(PlatformApplication.class);
        application.run(args);
    }
}
```
pom.xml 配置
```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
```
这种方式就可以使用IDE工具启动这个类的main方法运行或调试了，比较方便，启动整也很快。

## Spring Boot 项目打成War包放到Tomcat里运行
如果需要放到Tomcat下运行，这种就不行了。
需要再增加一个类继承SpringBootServletInitializer类。
```
public class ProductStartApplication extends SpringBootServletInitializer {
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(ProductApplication.class);
    }
}
```
此处的ProductApplication就是上个步骤中使用ProductApplication类，不需要做任何改动。

pom.xml也需要修改，在引入spring-boot-start-web中排除tomcat的相关依赖。并且加入servlet的相关包
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- 移除嵌入式tomcat插件 -->
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <!-- <version>3.1.0</version> -->
            <scope>provided</scope>
        </dependency>
```
这样就可以使用maven的打包命令打包后使用Tomcat运行了。使用IDE调试工具也正常。

## Spring Boot 项目启动自动运行
1、 CommandLineRunner 接口
Spring Boot 提供了一个接口CommandLineRunner，在Spring Boot 项目启动后自动运行这个接口的实现类，如果有多个实现类，还可以指定先后顺序，并且在实现类中正常使用Spring的功能，比如注入某个类、读取配置文件等。
```
@Component
@Order(value = 1)
public class PlatformRunner implements CommandLineRunner {

    /**
     * PlatformProperties是Spring boot 配置文件加载类。
     */
    @Autowired
    private PlatformProperties properties;

    @Override
    public void run(String... strings) throws Exception {
        System.out.println(properties.getPlatformName() + "-" + properties.getVersion());
    }
}

```
2、实现ApplicationListener 接口。
```
public class RunListener implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
        System.out.println("application start....");
    }
}

```
ApplictionListener接口的泛型参数有几种不同的事件:
1)ContextRefreshedEvent：当ApplicationContext初始化或者刷新时触发该事件。 
2)ContextClosedEvent：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。 
3)RequestHandleEvent：在Web应用中，当一个http请求（request）结束触发该事件。 
4)ContestStartedEvent：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。 
5)ContestStopedEvent：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。

## 自定义annotation简化配置
目前只是定义了一个annotation,并没有省去太多的配置，但只是一个简单的应用，可以进行一步挖掘项目的使用方法，进行相关的简化，达到统一配置的目的。
```
/**
 * 将自定义配置文件与启动类简化在一个annotation中，并且可以加入自定义的配置。
 * scanBasePackageClasses,这个配置项目是继承@SpringBootApplication的。
 **/
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootApplication
@PropertySource(value = {"classpath:platform.properties"})
@EnableConfigurationProperties(PlatformProperties.class)
public @interface PlatformApplication {

    @AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
    Class<?>[] scanBasePackageClasses() default {};
}

```
这样在其他引用此项目的其他SpringBoot 项目中就可以使用PlatformApplication这个注解来启动SpringBoot项目了。
