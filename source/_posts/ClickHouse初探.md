---
title: ClickHouse初探
date: 2024-06-29
tags: 
    - ClickHouse
    - Database
categories: Summary
copyright: true
---
:pushpin: 最近的项目用到了`ClickHouse`，全称`Click Stream, Data Warehouse`，是用于联机分析的列式数据库，由Russian的Yandex开发逐渐演进而来，在这里记录一些初次使用ClickHouse时摸索的一些特性和坑。

:dash::clock10::sleeping:

<!--more-->

## 1. 优缺点

### 1.1 Benefits

- ROLAP实时分析、实时查询、列式存储、完善的SQL支持、高可用，适合大数据量的数据分析，支持PB级别的数据量

### 1.2 Drawbacks

- 不支持事务
- join的语法不够完善
- 采用并行处理，不支持高并发
- 稀疏索引导致查询性能不稳定，删改数据性能较差

## 2. 应用场景

- 主要用于数据分析领域，适合OLAP场景，不适合OLTP场景

## 3. 安装

> 在Ubuntu的环境下用apt安装

- Use repository from Yandex

  - Add key

    ```shell
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E0C56BD4
    ```

    ```shell
    OutputExecuting: /tmp/apt-key-gpghome.JkkcKnBAFY/gpg.1.sh --keyserver keyserver.ubuntu.com --recv E0C56BD4

    gpg: key C8F1E19FE0C56BD4: public key "ClickHouse Repository Key <milovidov@yandex-team.ru>" imported
    gpg: Total number processed: 1
    gpg:               imported: 1
    ```

  - Add repository and update

    ```shell
    echo "deb http://repo.yandex.ru/clickhouse/deb/stable/ main/" | sudo tee /etc/apt/sources.list.d/clickhouse.list

    sudo apt update
    ```

- Install ClickHouse
    You need to enter the password for the `default` user when installiation in 75% progress.

    ```shell
    sudo apt install clickhouse-server clickhouse-client
    ```

- Start ClickHouse

    ```shell
    sudo service clickhouse-server start
    sudo service clickhouse-server status
    ```

## 4. 防火墙设置和远程连接

> 如果仅在同一服务器访问ClickHouse，则无需设置防火墙
> 如果需要远程访问，则需要配置用户权限

- 修改ClickHouse配置文件，监听所有IP地址
  - 取消注释，大约在配置文件的一百五十多行

    ```shell
    sudo vim /etc/clickhouse-server/config.xml
    ```

    ```xml
    <listen_host>::</listen_host>
    ```

- 重启ClickHouse

    ```shell
    systemctl restart clickhouse-server.service
    ```

- 远程访问

> 无法使用`default`用户远程访问，需要增加`default`的管理权限后创建新用户

- 修改`default`用户权限，取消`access_management>1</access_management>`的注释
  
    ```shell
    sudo vim /etc/clickhouse-server/users.xml
    ```

    ```xml
    <users>
        <default>
            <!-- ... -->
            <!-- ... -->
            <!-- ... -->
            <!-- User can create other users and grant rights to them. -->
            <!-- <access_management>1</access_management> -->
        </default>
    </users>
    ```

    ```shell
    systemctl restart clickhouse-server.service
    ```

- 创建只读权限和新用户

    ```shell
    clickhouse-client --password
    ```

    ```sql
    CREATE ROLE role_read;
    GRANT SELECT ON db.* TO role_read;

    CREATE USER user_read IDENTIFIED WITH sha256_password  BY 'password';

    GRANT role_read TO user_read;
    ```

- 访问

    ```shell
    clickhouse-client -m -u user_read --password 'your_password'
    ```

- DBeaver连接
  - Driver: ClickHouse
  - URL: jdbc:clickhouse://your_ip:8123
  - User: user_read
  - Password: your_password

- SpringBoot连接
  - Dependency

    ```xml
    <dependency>
        <groupId>ru.yandex.clickhouse</groupId>
        <artifactId>clickhouse-jdbc</artifactId>
        <version>0.2.4</version>
    </dependency>
    ```

  - Application.yml

    ```yaml
    spring:
      application:
        name: "logBase"
      profiles:
        active: "dev"
      main:
        allow-circular-references: true
      datasource:
        type: com.alibaba.druid.pool.DruidDataSource
        clickhouse:
        driverClassName: ${log-base.datasource.driver-class-name}
        url: jdbc:clickhouse://${log-base.datasource.host}:${log-base.datasource.port}/${log-base.datasource.database}
        username: ${log-base.datasource.username}
        password: ${log-base.datasource.password}
        testWhileIdle: ${log-base.datasource.test-while-idle}
        validationQuery: ${log-base.datasource.validation-query}
    ```

  - ClickHouseProperties

    ```java
    @Data
    @Component
    @ConfigurationProperties(prefix = "spring.datasource.clickhouse")
    public class ClickHouseProperties {

        private String driverClassName;

        private String url;

        private String username;

        private String password;

        private String validationQuery;

        private String testWhileIdle;
    }
    ```

  - CLickHouseConfiguration.java

    ```java
    @Configuration
    public class ClickHouseConfiguration {

        private final ClickHouseProperties clickHouseProperties;

        @Autowired
        public ClickHouseConfiguration(ClickHouseProperties clickHouseProperties) {
            this.clickHouseProperties = clickHouseProperties;
        }

        @Bean(name = "clickhouseDataSource")
        public DataSource dataSource() {
            log.info("Init clickhouse datasource");
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setUrl(clickHouseProperties.getUrl());
            dataSource.setDriverClassName(clickHouseProperties.getDriverClassName());
            dataSource.setUsername(clickHouseProperties.getUsername());
            dataSource.setPassword(clickHouseProperties.getPassword());
            dataSource.setTestWhileIdle(Boolean.parseBoolean(clickHouseProperties.getTestWhileIdle()));
            dataSource.setValidationQuery(clickHouseProperties.getValidationQuery());
            return dataSource;
        }
    }
    ```
