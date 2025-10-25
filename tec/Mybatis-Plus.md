# Mybatis Plus

>  [Mybatis Plus ](https://baomidou.com/introduce/) 

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作

- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错



## 引入依赖

### Spring Boot2

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.14</version>
</dependency>
```

### Spring Boot3

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.14</version>
</dependency>
```

### Spring Boot4 (自3.5.13开始)

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot4-starter</artifactId>
    <version>3.5.14</version>
</dependency>
```

### Spring

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus</artifactId>
    <version>3.5.14</version>
</dependency>
```



## 配置

配置 MapperScan 注解

```java
@SpringBootApplication
@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```



## Mapper 层常用 API

`insert(T entity)`：插入一条记录。

`deleteById(Serializable id)`：根据主键删除。

`deleteByMap(Map<String, Object> columnMap)`：根据 columnMap 条件删除。

`delete(Wrapper<T> wrapper)`：根据条件构造器删除。

`updateById(T entity)`：根据主键更新。

`update(T entity, Wrapper<T> updateWrapper)`：根据条件构造器更新。

`selectById(Serializable id)`：根据主键查询。

`selectBatchIds(Collection<? extends Serializable> idList)`：根据主键集合查询。

`selectByMap(Map<String, Object> columnMap)`：根据 Map 条件查询。

`selectOne(Wrapper<T> queryWrapper)`：根据条件查询单条数据。

`selectCount(Wrapper<T> queryWrapper)`：根据条件统计数量。

`selectList(Wrapper<T> queryWrapper)`：根据条件查询列表。

`selectPage(IPage<T> page, Wrapper<T> queryWrapper)`：分页查询。



## Wrapper

`eq("name", "Tom")`：等于。

`ne("status", 1)`：不等于。

`gt("age", 18)`：大于。

`lt("age", 30)`：小于。

`between("age", 18, 30)`：区间。

`like("name", "Tom")`：模糊匹配。

`orderByDesc("create_time")`：倒序排序。

`or()`：OR 连接条件。

`in("id", ids)`：IN 查询。
