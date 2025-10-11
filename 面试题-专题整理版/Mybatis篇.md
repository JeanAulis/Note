[toc]

---

## 1. mybatis的#和$的区别

**`#{}`占位符**

特点：

- 使用预编译PreparedStatement（不复用的话会导致性能差一些）
- 自动转译防止SQL注入
- 参数值会被加上单引号
- 适用于参数值

**`${}`占位符**

特点：

- 直接进行字符串替换(字符串拼接)
- 不会进行转译，存在SQL注入风险
- 参数值不会加上引号
- 适合动态表名、列名、order by等（多次请求性能高，且注入风险不高）



## 2. 介绍一下mybatis的一级和二级缓存

MyBatis是Java持久层框架，其缓存机制提升查询性能。

- **一级缓存**：SqlSession级别（默认开启）。同一SqlSession内，相同SQL和参数的查询结果缓存到缓存中。下次相同查询直接从缓存取。事务提交/关闭时清空。作用域小，适合短生命周期操作。
- **二级缓存**：Mapper级别（需手动开启，在mapper.xml配置`<cache>`）。跨SqlSession共享，存储在 PerpetualCache（默认HashMap，可用Ehcache/Redis替换）。查询先查二级、再一级、再数据库。更新操作会清空相关缓存。支持序列化，适合读多写少场景。

注意：一级缓存线程不安全，多线程需新SqlSession；二级缓存需配置eviction策略（如LRU）避免内存溢出。

>  [!important] 
>
>  类似：
>
>  ​	隔离级别：可重复读
>
>  可能会问：
>
>  1. `#`和`$`的区别
>  2. 问到底层原理则战略性放弃



## 2.（else）xml 映射文件中，除了常见的 select、insert、update、delete 标签之外，还有哪些标签？

还有很多其他的标签， `<resultMap>`、 `<parameterMap>`、 `<sql>`、 `<include>`、 `<selectKey>` ，加上动态 sql 的 9 个标签， `trim|where|set|foreach|if|choose|when|otherwise|bind` 等，其中 `<sql>` 为 sql 片段标签，通过 `<include>` 标签引入 sql 片段， `<selectKey>` 为不支持自增的主键生成策略标签。



## 3. （else）MyBatis 执行批量插入，能返回数据库主键列表吗？

能，JDBC 都能，MyBatis 当然也能。



## 4. （else）MyBatis 是如何进行分页的？分页插件的原理是什么？

**(1)** MyBatis 使用 RowBounds 对象进行分页，它是针对 ResultSet 结果集执行的内存分页，而非物理分页；**(2)** 可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，**(3)** 也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。

举例：`select _ from student` ，拦截 sql 后重写为：`select t._ from （select \* from student）t limit 0，10`



## 5. （else）MyBatis 动态 sql 是做什么的？都有哪些动态 sql？能简述一下动态 sql 的执行原理不？

MyBatis 动态 sql 可以让我们在 xml 映射文件内，以标签的形式编写动态 sql，完成逻辑判断和动态拼接 sql 的功能。其执行原理为，使用 OGNL 从 sql 参数对象中计算表达式的值，根据表达式的值动态拼接 sql，以此来完成动态 sql 的功能。

MyBatis 提供了 9 种动态 sql 标签:

- `<if></if>`
- `<where></where>(trim,set)`
- `<choose></choose>（when, otherwise）`
- `<foreach></foreach>`
- `<bind/>`

- [ ] 执行原理



## 来源

---

[JavaGuide](https://github.com/Snailclimb/JavaGuide/blob/main/docs/system-design/framework/mybatis/mybatis-interview.md)