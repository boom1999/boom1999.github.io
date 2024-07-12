---
title: application yml配置
date: 2024-04-12
tags: 
    - SprinBoot
    - application
categories: Summary
copyright: true
---

> `SpringBoot`常用配置文件`application.properties`或`application.yml`进行全局配置，`start.spring.io`的默认是用`properties`，只要改成`yml`文件即可，个人认为语法更舒适，在这篇记录一些常用的设置以及常用的一些包的参数，以便后续开发过程中及时翻阅，持续更新。

<!--more-->

## :books: 1. application配置文件

### 1.1. 全局配置文件

> 启动类下的全局配置，修改`SpringBoot`在底层的默认值

- `application.properties`
- `application.yml`

### 1.2. Proterties和YML以及XML区别

- `Properties`

    ``` properties
    server.port=8080
    ```

- `YAML`

    ``` YAML
    server:
      port: 8080
    ```

- `XML`

    ``` XML
    <server>
        <port>8080</port>
    </server>
    ```

## :satellite: 2. YAML语法

### 2.1. 基本语法

- KV键值对，空格缩进（一般2格），左对齐的为同一层级
- 属性和值大小写敏感，值不能空着

### 2.2. 字面量

- 字符串，默认不需要使用引号
  - 单引号，不会转义特殊字符例如'aaa \n bbb'不会输出换行
  - 双引号，会转义特殊字符例如"aaa \n bbb"会输出换行
- 布尔，`True`或`true`，`Flase`或`false`
- 整数，`123`或`0b1010_0111_0100_1010_1110`
- 浮点数，`0.123`或者科学计数`6.85e+5`
- `NULL`，可用`~`表示`null`
- 日期，2024-07-12`，日期使用ISO 8601格式，yyyy-MM-dd
- 时间，`2024-07-12T14:31:00+08:00`，时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区

### 2.3. 对象

- 普通的缩进表示层级联系

    ```yml
    redis:
      host: 192.168.1.111
      port: 6379
      password: 123456
    ```

- 行内写法

    ```yml
    redis: {host: 192.168.1.111,port: 6379,password: 123456}
    ```

### 2.4. 数组

- 普通写法，用`-`表示

    ``` yml
    persons:
      - zhangsan
      - lisi
      - wangwu
    ```

- 行内写法，用`[]`

    ```yml
    persons: [zhangsan,lisi,wangwu]
    ```

### 2.5. 锚点

- 用`&`建立锚点，用`<<`合并数据，用`*`引用锚点内容

    ``` yml
    defaults: &defaults
      host:  192.168.1.129
      port:  8080

    dev:
      database: dev_db
      <<: *defaults

    prod:
      database: prod_db
      <<: *defaults

    # 相当于
    # defaults: 
    #   host:  192.168.1.111
    #   port:  6379
    
    # dev:
    #   database: dev_db
    #   host:  192.168.1.111
    #   port:  6379

    # prod:
    #   database: prod_db
    #   host:  192.168.1.111
    #   port:  6379
    ```

- 直接在键后设置锚点，加空格后紧跟值，锚点名自定义，引用同样用`*`

    ``` yml
    defaults: 
      host:  &host 192.168.1.129
      port:  &port 8080
    
    dev:
      database: dev_db
      host:  *host
      port:  *port

    prod:
      database: prod_db
      host:  *host
      port:  *port
    ```

## :mag_right: 3. 配置文件注入

- 一般用注解`@ConfigurationProperties(prefix = "async.executor.thread")`，加上`@Data`注解或自己写`getter`和`setter`方法
  - 配合`@Component`定义组件，来定义配置数据的结构Properties
  - 再配合`@Configuration`使用，用于定义配置类，再在配置类中注入配置文件的结构Properties，当然也可合并成一步
- 只有容器中的组件，才能提供`@ConfigurationProperties`功能
- `prefix = "xxx.xxx"`是映射该路径下的配置文件，这里定义好`getter`和`setter`方法就能直接调用`setKeepAliveSeconds(int keepAliveSeconds)`等方法

    ``` yml
    async:
    executor:
        thread:
        keepAliveSeconds: 60
        namePrefix: async-service-
    ```

- 可能需要`@ComponentScan(basePackages = {"com.***", "com.***"})`，扫描组件所在的`module`

- 可选导入配置文件冲处理器，这样会有提示

    ``` xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    ```

- 可选用@Value配置

  - 若要在某个业务逻辑中要用到配置的值，可以用`@Value`，若已经整体设计了配置类的组件，则建议用`@ConfigurationProperties`批量注入

    ``` java
    @Value("${thread.keepAliveSeconds}")
    private int keepAliveSeconds;
    ```
  
  - 不支持松散绑定(Relaxed bindiing)，批量注入支持，`firstName`和`first-name`和`first_name`
  - 支持SPEL，批量注入不支持
  - 不持支map、对象等复杂类型以及JSR303校验

## :zzz: 4. 多环境支持

- `application-{profile}.properties/yml`在测试环境和生成环境用不同的配置，包括一些`id`和`key`等重要的信息切换，例如`application-dev.yml`和`application-prod.yml`，再用`${xxx.xxx.driver-class-name}`在主配置文件中引入不同环境的配置参数
- `properties`优先级大于同级别的`yml`，要么删掉要么确保内有相同内容被指定
- 指定测试或生产环境，可以在`application.yml`中指定，或者打包jar时配置启动参数`Program arguments`为`--spring.profiles.active=dev`或者配置虚拟机参数`VM options`为`-Dspring.profiles.active=dev`

    ``` yml
    spring:
      profiles:
        active: "dev"
    ```

- 加载顺序

    ``` md
    –file:./config/（根目录下的config文件夹）

    –file:./（根目录下）

    –classpath:/config/（resources/config）

    –classpath:/（resources）
    ```

- 启动类开启`@EnableAutoConfiguration`（`@SpringBootApplication`已经引入，不用重复）
