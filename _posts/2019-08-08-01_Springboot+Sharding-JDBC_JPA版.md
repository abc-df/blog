---
title: Springboot+Sharding-JDBC_JPA
tags: java
key: 
modify_date: 2019-08-08 20:00:00 +08:00
---

# Sharding-JDBC
Sharding的基本思想就要把一个数据库切分成多个部分放到不同的数据库(server)上，从而缓解单一数据库的性能问题。不太严格的讲，对于海量数据的数据库，如果是因为表多而数据多，这时候适合使用垂直切分，即把关系紧密（比如同一模块）的表切分出来放在一个server上。如果表并不多，但每张表的数据非常多，这时候适合水平切分，即把表的数据按某种规则（比如按ID散列）切分到多个数据库(server)上。当然，现实中更多是这两种情况混杂在一起，这时候需要根据实际情况做出选择，也可能会综合使用垂直与水平切分，从而将原有数据库切分成类似矩阵一样可以无限扩充的数据库(server)阵列。

 <!--more-->
# 常用的分库分表中间件
简单易用的组件：

    当当sharding-jdbc
    蘑菇街TSharding

强悍重量级的中间件：

    sharding
    TDDL Smart Client的方式（淘宝）
    Atlas(Qihoo 360)
    alibaba.cobar(是阿里巴巴（B2B）部门开发)
    MyCAT（基于阿里开源的Cobar产品而研发）
    Oceanus(58同城数据库中间件)
    OneProxy(支付宝首席架构师楼方鑫开发)
    vitess（谷歌开发的数据库中间件）

# Springboot+Sharding-JDBC_JPA

建表sql:

```sql
CREATE DATABASE database0;
USE database0;
DROP TABLE IF EXISTS `goods_0`;
CREATE TABLE `goods_0` (
  `goods_id` bigint(20) NOT NULL,
  `goods_name` varchar(100) COLLATE utf8_bin NOT NULL,
  `goods_type` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`goods_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
DROP TABLE IF EXISTS `goods_1`;
CREATE TABLE `goods_1` (
  `goods_id` bigint(20) NOT NULL,
  `goods_name` varchar(100) COLLATE utf8_bin NOT NULL,
  `goods_type` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`goods_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
CREATE DATABASE database1;
USE database1;
DROP TABLE IF EXISTS `goods_0`;
CREATE TABLE `goods_0` (
  `goods_id` bigint(20) NOT NULL,
  `goods_name` varchar(100) COLLATE utf8_bin NOT NULL,
  `goods_type` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`goods_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

DROP TABLE IF EXISTS `goods_1`;
CREATE TABLE `goods_1` (
  `goods_id` bigint(20) NOT NULL,
  `goods_name` varchar(100) COLLATE utf8_bin NOT NULL,
  `goods_type` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`goods_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
```

分库：

    根据数据库表中字段goods_id的大小进行判断，如果goods_id大于20则使用database0，否则使用database1。

分表：

    根据数据库表中字段goods_type的数值的奇偶进行判断，奇数使用goods_1表，偶数使用goods_0表。

# 代码

## pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.dalaoyang</groupId>
    <artifactId>springboot2_shardingjdbc_fkfb</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot2_shardingjdbc_fkfb</name>
    <description>springboot2_shardingjdbc_fkfb</description>

    <properties>
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
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
        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.9</version>
        </dependency>
        <!-- sharding-jdbc -->
        <dependency>
            <groupId>com.dangdang</groupId>
            <artifactId>sharding-jdbc-core</artifactId>
            <version>1.5.4</version>
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
## 配置文件

```java
##Jpa配置
spring.jpa.database=mysql
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none

##数据库配置
##数据库database0地址
database0.url=jdbc:mysql://localhost:3306/database0?characterEncoding=utf8&useSSL=false
##数据库database0用户名
database0.username=root
##数据库database0密码
database0.password=admin
##数据库database0驱动
database0.driverClassName=com.mysql.jdbc.Driver
##数据库database0名称
database0.databaseName=database0

##数据库database1地址
database1.url=jdbc:mysql://localhost:3306/database1?characterEncoding=utf8&useSSL=false
##数据库database1用户名
database1.username=root
##数据库database1密码
database1.password=admin
##数据库database1驱动
database1.driverClassName=com.mysql.jdbc.Driver
##数据库database1名称
database1.databaseName=database1

```
## 实体

```java

@Entity
@Table(name="goods")
@Data
public class Goods {
    @Id
    private Long goodsId;

    private String goodsName;

    private Long goodsType;
}

```
## 数据库配置

```java

@Data
@ConfigurationProperties(prefix = "database0")
@Component
public class Database0Config {
    private String url;
    private String username;
    private String password;
    private String driverClassName;
    private String databaseName;

    public DataSource createDataSource() {
        DruidDataSource result = new DruidDataSource();
        result.setDriverClassName(getDriverClassName());
        result.setUrl(getUrl());
        result.setUsername(getUsername());
        result.setPassword(getPassword());
        return result;
    }
}

```


```java

@Data
@ConfigurationProperties(prefix = "database1")
@Component
public class Database1Config {
    private String url;
    private String username;
    private String password;
    private String driverClassName;
    private String databaseName;

    public DataSource createDataSource() {
        DruidDataSource result = new DruidDataSource();
        result.setDriverClassName(getDriverClassName());
        result.setUrl(getUrl());
        result.setUsername(getUsername());
        result.setPassword(getPassword());
        return result;
    }
}

```

## DataSourceConfig用于创建数据源和使用分库分表策略，其中分库分表策略会调用分库算法类和分表算法类

```java

@Configuration
public class DataSourceConfig {

    @Autowired
    private Database0Config database0Config;

    @Autowired
    private Database1Config database1Config;

    @Autowired
    private DatabaseShardingAlgorithm databaseShardingAlgorithm;

    @Autowired
    private TableShardingAlgorithm tableShardingAlgorithm;

    @Bean
    public DataSource getDataSource() throws SQLException {
        return buildDataSource();
    }

    private DataSource buildDataSource() throws SQLException {
        //分库设置
        Map<String, DataSource> dataSourceMap = new HashMap<>(2);
        //添加两个数据库database0和database1
        dataSourceMap.put(database0Config.getDatabaseName(), database0Config.createDataSource());
        dataSourceMap.put(database1Config.getDatabaseName(), database1Config.createDataSource());
        //设置默认数据库
        DataSourceRule dataSourceRule = new DataSourceRule(dataSourceMap, database0Config.getDatabaseName());

        //分表设置，大致思想就是将查询虚拟表Goods根据一定规则映射到真实表中去
        TableRule orderTableRule = TableRule.builder("goods")
                .actualTables(Arrays.asList("goods_0", "goods_1"))
                .dataSourceRule(dataSourceRule)
                .build();

        //分库分表策略
        ShardingRule shardingRule = ShardingRule.builder()
                .dataSourceRule(dataSourceRule)
                .tableRules(Arrays.asList(orderTableRule))
                .databaseShardingStrategy(new DatabaseShardingStrategy("goods_id", databaseShardingAlgorithm))
                .tableShardingStrategy(new TableShardingStrategy("goods_type", tableShardingAlgorithm)).build();
        DataSource dataSource = ShardingDataSourceFactory.createDataSource(shardingRule);
        return dataSource;
    }


    @Bean
    public KeyGenerator keyGenerator() {
        return new DefaultKeyGenerator();
    }

}
```
## 分库分表算法

由于这里只是简单的分库分表样例，所以分库类这里实现SingleKeyDatabaseShardingAlgorithm类，采用了单分片键数据源分片算法，需要重写三个方法，分别是：

doEqualSharding：SQL中==的规则。
doInSharding：SQL中in的规则。
doBetweenSharding：SQL中between的规则。
本文分库规则是基于值大于20则使用database0，其余使用database1，所以简单if，else就搞定了，分库算法类DatabaseShardingAlgorithm代码如下所示。

```java

/**
 * 这里使用的都是单键分片策略
 * 示例分库策略是：
 * GoodsId<=20使用database0库
 * 其余使用database1库
 */
@Component
public class DatabaseShardingAlgorithm implements SingleKeyDatabaseShardingAlgorithm<Long> {

    @Autowired
    private Database0Config database0Config;

    @Autowired
    private Database1Config database1Config;

    @Override
    public String doEqualSharding(Collection<String> availableTargetNames, ShardingValue<Long> shardingValue) {
        Long value = shardingValue.getValue();
        if (value <= 20L) {
            return database0Config.getDatabaseName();
        } else {
            return database1Config.getDatabaseName();
        }
    }

    @Override
    public Collection<String> doInSharding(Collection<String> availableTargetNames, ShardingValue<Long> shardingValue) {
        Collection<String> result = new LinkedHashSet<>(availableTargetNames.size());
        for (Long value : shardingValue.getValues()) {
            if (value <= 20L) {
                result.add(database0Config.getDatabaseName());
            } else {
                result.add(database1Config.getDatabaseName());
            }
        }
        return result;
    }

    @Override
    public Collection<String> doBetweenSharding(Collection<String> availableTargetNames,
                                                ShardingValue<Long> shardingValue) {
        Collection<String> result = new LinkedHashSet<>(availableTargetNames.size());
        Range<Long> range = shardingValue.getValueRange();
        for (Long value = range.lowerEndpoint(); value <= range.upperEndpoint(); value++) {
            if (value <= 20L) {
                result.add(database0Config.getDatabaseName());
            } else {
                result.add(database1Config.getDatabaseName());
            }
        }
        return result;
    }
}

```


```java
**
 * 这里使用的都是单键分片策略
 * 示例分表策略是：
 * GoodsType为奇数使用goods_1表
 * GoodsType为偶数使用goods_0表
 */
@Component
public class TableShardingAlgorithm implements SingleKeyTableShardingAlgorithm<Long> {

    @Override
    public String doEqualSharding(final Collection<String> tableNames, final ShardingValue<Long> shardingValue) {
        for (String each : tableNames) {
            if (each.endsWith(shardingValue.getValue() % 2 + "")) {
                return each;
            }
        }
        throw new IllegalArgumentException();
    }

    @Override
    public Collection<String> doInSharding(final Collection<String> tableNames, final ShardingValue<Long> shardingValue) {
        Collection<String> result = new LinkedHashSet<>(tableNames.size());
        for (Long value : shardingValue.getValues()) {
            for (String tableName : tableNames) {
                if (tableName.endsWith(value % 2 + "")) {
                    result.add(tableName);
                }
            }
        }
        return result;
    }

    @Override
    public Collection<String> doBetweenSharding(final Collection<String> tableNames,
                                                final ShardingValue<Long> shardingValue) {
        Collection<String> result = new LinkedHashSet<>(tableNames.size());
        Range<Long> range = shardingValue.getValueRange();
        for (Long i = range.lowerEndpoint(); i <= range.upperEndpoint(); i++) {
            for (String each : tableNames) {
                if (each.endsWith(i % 2 + "")) {
                    result.add(each);
                }
            }
        }
        return result;
    }
}
```
## Repository

```java
public interface GoodsRepository extends JpaRepository<Goods, Long> {

    List<Goods> findAllByGoodsIdBetween(Long goodsId1,Long goodsId2);

    List<Goods> findAllByGoodsIdIn(List<Long> goodsIds);
}
```
## 启动类

```java

@SpringBootApplication
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
@EnableTransactionManagement(proxyTargetClass = true)
@EnableConfigurationProperties
public class BaasApplication {


    public static void main(String[] args) {
        SpringApplication.run(BaasApplication.class, args);
    }

}

```








