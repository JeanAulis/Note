[toc]

---

## 1. 你说一下什么aop？你说一下实现aop的步骤？aop的底层实现原理是什么?





## 2. 你在项目中用aop实现什么功能？用的是什么通知？





## 3. 如何快速搭建一个springboot项目的步骤，还有你们用的springboot的版本是什么？

有好几种方法，一般是用第二种

### 第一种：Spring Initializr

- 可以使用Spring Initializr（通过连接 https://start.spring.io/ 来快速生成），通过可视化界面来快速搭建一个SpringBoot项目，优点是快速，低代码；缺点是可提供的配置项不完整，还需要手动引入依赖。

### 第二种

1. 创建Maven项目
2. 在Pom.xml中引入``spring-boot-starter-parent`和`spring-boot-starter-web`等Maven的GAV坐标
3. 写一个SpringBoot的配置文件application.yml
4. 创建启动类文件，加上@SringBootApplication注解，还有main方法实现SpringBootApplication.run(类名.class,args)方法

常用的版本是2.7.x，因为兼容Java8/11



## 4. mybatis的#和$的区别

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



## 5. 你说一下前后端怎么实现上传文件的步骤

前端建立一个`form`表单，使用`<input type="file">`来选择文件，再通过`XMLHttpRequest`或者`fetch`来提交`POST`请求（以后学到进度条了可以加进去，大文件跟踪上传进度）

后端使用`@RequestParam MultipartFile file`类型来接收文件

后端再选择保存到本地还是阿里的OSS，如果上传到阿里的OSS的话，就需要进行额外的配置（获取 AccessKey）Bucket，域名等配置



## 6. springboot的优点是什么？说一下SpringBoot自动装配原理

优点是：

自动装配：会自动装配Bean对象和配置类到SpringIoc容器，省去配置

起步依赖：引入起步依赖之后，通过Maven的依赖传递，将它需要的依赖也引入



自动装配的核心思想是**“约定大于配置”**，根据依赖自动导入配置好的相关配置

>  [!Note]
>
> 第三方的依赖只需要META-INF中的spring.factories文件里声明这些类的全类路径，在项目的启动过程中就可以自动的找到这些配置文件解析它们，交给IOC容器管理

SpringBoot的自动装配原理有三个核心部分：启动注解，自动装配入口和条件装配

1. SpringBoot提供了一个入口注解`@SpringBootApplication`，这个注解的功能是标记配置，开启自动装配，启用组件扫描（META-INF/spring.factories）
2. `@EnableAutoConfiguration`读取配置文件，获取配置列表，筛选配置类
3. `@ConditionalOnClass`根据配置决定哪些装配哪些Bean



## 7. 你们项目是怎么处理异常的？你说一下怎么实现全局自定义异常处理器

我们项目使用全局异常处理器

>  [!tip]
>
>  也可以使用AOP进行处理异常

通过两个核心注解来实现，一个是`@RestControllerAdvice`和`@Exceptionhandler`这两个注解来实现

首先创建一个 `GlobalExceptionHandler`类(普通Java类)，用`@RestControllerAdvice`标记为全局异常处理器

再使用`@Exceptionhandler`来定义要处理的异常方法



## 8. 你说一下过滤器和拦截器的区别

> - 过滤器（Filter）：当有一堆请求，只希望符合预期的请求进来。
>
> - 拦截器（Interceptor）：想要干涉预期的请求。
>
> - 监听器（Listener）：想要监听这些请求具体做了什么。

 可以从细粒度，实现和技术栈，作用范围

过滤器(Filter)是基于Servlet规范，FilterChain，责任链设计。拦截进入Selrvlet但未进入Spring容器的请求，可以拦截到方法的请求和响应，常用来做：过滤敏感词汇（防止sql注入），设置字符编码，URL级别的权限访问控制，压缩响应信息等。

拦截器(Interceptor)基于Spring AOP的思想实现，只拦截Controller的请求，需要重写`HandlerInterceptor`接口可以在方法执行前(preHandle)，执行后(afterCompletion)进行操作，还有回调操作(postHandle)。常用来做：日志记录，性能监控。

>   [!caution]
>
>  只有 preHandle 返回 true 的话，其他两个方法才会执行。

先经过过滤器才会到拦截器

>  [!tip]
>
> 使用**过滤器**：当你的功能与 Spring 无关，且需要拦截所有请求（包括静态资源）时，比如编码格式转换、统一的日志记录、跨域设置等。
>
> 使用**拦截器**：当你的功能与 Spring **业务**强相关，且需要精确到 Controller 方法执行时，比如用户登录状态校验、权限控制、请求耗时统计等。
>
> 过滤器是**外围**的、更底层的处理，而拦截器是**内部**的、更精细的业务处理。在 Spring 项目中，处理业务逻辑相关的增强，通常都推荐使用拦截器。



## 9. 你们是怎么保存用户登录信息的？你们的jwt需要在服务端存储吗？如果jwt过期了用户不就要重新登录了吗？你们怎么处理的？

使用JWT来保存信息

>  [!tip]
>
> 还有Seesion，cookie等

不需要在服务器进行存储

>  [!tip]
>
> 保存在客户端，每次请求把Token放在HTTP请求头(无状态Stateless)，JWT由三部分组成：头部(header)描述签名算法和了类型，载荷(payload)存放用户信息和声明，还有签名(signature)部分，用来验证消息是否被篡改
>
> ```css
> header.payload.signature
> ```

是的，jwt过期后就无法需要重新登陆，我们使用双token校验的方法来解决频繁重复的问题

>   [!tip]
>
>  登陆成功之后返回两个token给客户端，一个访问令牌（Access Token）和一个刷新令牌（Refresh Token）
>
>  - Access Token：一般有效期比较短，用来访问受保护的资源。
>
>  - Refresh Token：有效期很长，比如7天，用来在Access Token过期后获取新的Access Token。
>
>  如果是关于登陆认证（比当前问题范围大一些），需要回答双token三认证：基于双token，如果用户退出登陆，马上把Refresh token加入Redis的黑名单，再次登陆时先检查是否在黑名单，如果不在，判断它是否过期，如果一切正常，才下发Access Token



## 10. 说一下jwt由哪些部分组成？说一下jwt为什么可以防篡改和检查过时？

JWT由三部分组成：头部(header)描述签名算法和类型，载荷(payload)存放用户信息和声明，还有签名(signature)部分，用来验证消息是否被篡改

```css
header.payload.signature
```

签名部分会校验是否被篡改，载荷(payload)里面保存了过期时间戳，签名部分保证了载荷的内容可信，所以这个过期时间戳可信



## 11. hash算法可逆吗？什么是加盐？md5和Bcrypt的区别？签名为什么可以解决防篡改的问题？

不可逆。



加盐是在对密码Hash之前添加**随机且唯一**的字符串（盐值），解决彩虹表攻击和相同密码识别，盐值通常和Hash值一起存储，不需要保密。

- MD5需要手动实现加盐，而且生成Hash值快速


- Bcrypt实现了自动加盐，有一个可调整的工作因子



和JWT防篡改类似



>  [!tip]
>
> 为什么不可逆？
>
> 信息压缩：输出的哈希值都是固定的，也就是大量信息被丢弃，输出的值只能验证原来的内容是否被修改过，但是不能还原出原来完整的内容。
>
> 雪崩效应：哈希算法对数据敏感，微小的改动也会导致最终的哈希值发生巨大变化，很难方向推导。
>
> 哈希碰撞：即使存在两个不同的输入，它们输出的哈希值相同，这种概率非常的小（MD5有约1.8×10^19^个值，SHA-256有约3.4×10^38^个值）虽然小，但是也存在，所以无法确定唯一的输入

>  [!tip]
>
> 盐值在注册时生成随机盐值（可以集合用户名，时间戳来生成），通过密码+盐值进行Hash，最后存储盐值和Hash
>
> ```tex
> 数据库存储格式：
> 用户ID | 盐值 | 密码Hash值
> 1001 | aB3$kL9x | hash(password+aB3$kL9x)
> 1002 | mN7@pQ2z | hash(password+mN7@pQ2z)
> ```





## 12. 你们有做单元测试吗？怎么做单元测试的？





## 13. @RestController和@Controller的区别

**@Controller**：

- 是Spring MVC框架的一个注解
- 用于定义一个控制器类，返回视图（JSP、HTML等）。
- 方法返回的是视图名称，通常与 ModelAndView 配合使用来渲染视图。



**@RestController**：

- Spring 4.0 的一个组合注解
- 继承了 @Controller 并且包含 @ResponseBody，用于直接返回对象数据，通常是 JSON 或 XML 格式。
- 适用于构建 RESTful 风格的 Web 服务接口。



## 14. 什么是IOC和DI？

- **IOC（Inversion of Control，控制反转）**：

  - IOC 是一种设计思想，将对象的创建和管理交给 Spring 容器，而不是由类自己控制。这样可以实现松耦合、灵活的依赖管理。

  

- **DI（Dependency Injection，依赖注入）**：

  - DI 是一种实现 IOC 的技术，指将对象所依赖的其他对象（依赖项）注入到它的构造函数、字段或方法中，而不是由对象自己去创建这些依赖。

  - 依赖注入有三种主要方式：

    1. **构造函数注入**：通过构造函数传递依赖对象。
    2. **字段注入**：通过字段直接注入依赖对象。
    3. **方法注入**：通过 setter 方法注入依赖对象。




## 15. @Autowired和@Resource的区别

1. 来源不同
   1. @Resource 是 Java EE（Java Platform, Enterprise Edition）规范定义的注解，位于 javax.annotation 包中，不仅可以用于 Spring 环境，还可以用于其他 Java EE 容器。
   2. @Autowired 是 Spring 框架定义的注解，位于 org.springframework.beans.factory.annotation 包中，主要用于 Spring 环境中。
2. 注入方式不同
   1. @Resource默认按名称注入，如果在spring容器找不到对应名称的 Bean，则按照 类型 进行注入。
   2. @Autowired默认按照 类型 的方式进行注入。如果有多个类型相同的 Bean，可以结合 @Qualifier 使用指定具体的 Bean 名称。也可以使用@primary设置优先级来注入，如果两个都存在，优先匹配@Qualifier。



## 16. 什么是代理？静态代理和动态代理的区别是什么？jdk动态代理和cglib的区别是什么？

代理是指一个对象（代理对象）代替另一个对象（目标对象）来执行操作，控制对目标对象的访问

静态代理是在编译阶段生成代理类，而动态代理是在程序运行时动态生成代理类。

~~Spring AOP中的~~  动态代理主要有两种方式，JDK动态代理和CGLIB动态代理：

- JDK动态代理**要求目标对象必须实现接口**，它是基于接口的代理，无法直接代理没有实现接口的类
- CGLIB是通过继承的方式做的动态代理 , 如果某个类被标记为final，那么它是无法使用 CGLIB做动态代理的。Enhancer.create(父类的字节码对象, 代理执行器)

> [!tip]
>
> 他们都不可以对final修饰的类进行代理



## 17. get请求和post请求的区别？

GET请求一般用于查询,POST请求一般用于提交数据; 

GET的安全性低因为参数值会暴露在地址栏,POST安全性相对较好一些;

GET带参数可能会受浏览器器地址栏的长度限制,POST的参数长度没有限制。



## 18. 说一下你理解的spring？DI注入有几种方式？推荐用哪种DI？

Spring是一个以IoC和AOP为核心的轻量级容器框架，目的是简化企业级Java开发。

- IoC/DI：控制反转和依赖注入，将对象创建的控制权交给容器，DI依赖注入是IoC的实现方式，通过容器管理Bean的依赖关系。
- AOP：面向切面编程，无侵入增强方法，
- 模块化设计：Spring 提供多个模块，如 Spring MVC（Web 开发）、Spring Data（数据访问）、Spring Security（安全）、Spring Boot（快速开发）等，



Spring的DI注入主要有三种方式：

1. 构造函数注入：通过类的构造函数注入，适合强依赖，保证对象不可变性和完整性。
2. Setter注入：通过Setter方法注入，适合可选依赖或需动态更新的场景。
3. 字段注入：直接在字段上加`@Autowired`，虽然方便但隐藏依赖，不推荐生产代码。
4. 还有方法注入

在实际项目中，我们优先使用构造函数注入核心依赖，用Setter注入可选配置，避免字段注入以保证代码可测试性。

>  [!important]
>
> 可能会问：
>
> 1. 关于AOP？
>
> 2. 什么是IoC/DI？
>
> 3. ~~自动装配原理~~
>
> 4. ---
>
> 5. Bean 生命周期
>
> 6. 循环依赖



## 19. 说一下bean的生命周期

- 分为实例化、属性赋值、初始化、销毁这4个大阶段;

- 再是初始化的具体操作,有Aware接口的依赖注入、BeanPostProcessor在初始化前后的处理以及InitializingBean和init-method的初始化操作;

- 销毁的具体操作,有注册相关销毁回调接口,最后通过DisposableBean和destory-method进行销毁。

Bean生命周期的主要阶段

1. **实例化（Instantiation）**：Spring 容器通过反射机制调用构造函数，创建 Bean 的空实例对象。
2. **属性赋值（Populate Properties）**：Spring 根据配置或注解，将 Bean 的属性和依赖注入其中。
3. **初始化前处理（Pre-Initialization）**：调用 `BeanPostProcessor` 的 `postProcessBeforeInitialization()` 方法进行前置处理。
4. **初始化（Initialization）**：执行 `@PostConstruct`、`InitializingBean` 或 `init-method` 等方法，完成 Bean 的初始化。
5. **初始化后处理（Post-Initialization）**：调用 `BeanPostProcessor` 的 `postProcessAfterInitialization()` 方法进行后置处理，通常用于生成代理对象。
6. **使用阶段（In Use）**：Bean 已经准备好，被应用程序获取并使用。
7. **销毁前处理（Pre-Destruction）**：在容器关闭时，调用 `@PreDestroy`、`DisposableBean` 或 `destroy-method` 等方法来销毁 Bean。

>  [!important] 
>
> 可能会问:
>
> 1. Bean实例化的三种方法（静态工厂，实例工厂，构造函数）



## 20. spring怎么解决循环依赖？spring框架有帮我们自动解决构造注入的循环依赖问题吗？为什么框架也不能自动解决？

### Spring如何解决循环依赖

- **什么是循环依赖** 循环依赖是指在软件系统中，两个或多个组件相互依赖，形成一个闭环的依赖关系。

  **Spring如何解决循环依赖** Spring主要通过**三级缓存**机制来解决循环依赖：

  1. **一级缓存(singletonObjects)** - 单例池，缓存已经经历了完整生命周期、已经初始化完成的Bean对象
  2. **二级缓存(earlySingletonObjects)** - 缓存早期的Bean对象（生命周期还没有走完的半成品Bean）
  3. **三级缓存(singletonFactories)** - 缓存ObjectFactory对象工厂，用来创建某个对象

  **详细解决过程：**

  1. **实例化A对象**：先实例化A对象，同时创建ObjectFactory对象存入三级缓存singletonFactories
  2. **A初始化需要B**：A在初始化时需要注入B对象，触发B的创建逻辑
  3. **实例化B对象**：B实例化完成，也会创建ObjectFactory对象存入三级缓存singletonFactories
  4. **B需要注入A**：B初始化时需要注入A，通过三级缓存中的ObjectFactory生成A对象并存入二级缓存（可能是A的普通对象或代理对象）
  5. **B创建完成**：B从二级缓存earlySingletonObjects获得A对象后完成注入，B创建成功并存入一级缓存singletonObjects
  6. **A创建完成**：回到A对象初始化，因为B对象已创建完成，直接注入B，A创建成功存入一级缓存singletonObjects
  7. **清理临时对象**：清除二级缓存中的临时A对象

- B创建完成后，A也能完成创建

### 构造注入的循环依赖问题

**Spring框架无法自动解决构造注入的循环依赖**，原因如下：

### 为什么不能解决？

1. **时机问题** - 构造注入发生在对象实例化时，此时对象还未创建完成，无法提前暴露
2. **依赖顺序** - 构造注入要求在创建对象时就必须提供所有依赖，形成死锁
3. **技术限制** - Java对象必须先通过构造函数创建实例，才能进行后续操作



>  [!note]
>
> beanfactory和factorybean的区别





## 21. 说一下SpringMVC的执行流程

### 1. 请求接收阶段

用户发送HTTP请求到服务器，请求首先被**DispatcherServlet**（前端控制器）接收。DispatcherServlet是SpringMVC的核心组件，负责统一处理所有的请求。

### 2. 处理器映射阶段

DispatcherServlet调用**HandlerMapping**（处理器映射器）来查找能够处理当前请求的Handler（通常是Controller中的某个方法）。HandlerMapping会根据请求的URL、请求方法等信息来确定具体的处理器。

### 3. 处理器适配阶段

找到Handler后，DispatcherServlet通过**HandlerAdapter**（处理器适配器）来执行具体的Handler。HandlerAdapter的作用是适配不同类型的Handler，使得DispatcherServlet能够以统一的方式调用各种Handler。

### 4. 业务处理阶段

HandlerAdapter调用具体的**Controller**方法执行业务逻辑。Controller处理完业务逻辑后，通常会返回一个ModelAndView对象，其中包含了视图名称和模型数据。

### 5. 视图解析阶段

DispatcherServlet将ModelAndView传递给**ViewResolver**（视图解析器）。ViewResolver根据视图名称解析出具体的View对象，比如JSP页面、Thymeleaf模板等。

### 6. 视图渲染阶段

**View**对象负责将模型数据渲染成最终的响应内容。View会将Controller传递的数据填充到模板中，生成完整的HTML页面或其他格式的响应。

### 7. 响应返回阶段

最终的响应内容通过DispatcherServlet返回给客户端浏览器。





## 22. 介绍一下mybatis的一级和二级缓存

MyBatis是Java持久层框架，其缓存机制提升查询性能。

- **一级缓存**：SqlSession级别（默认开启）。同一SqlSession内，相同SQL和参数的查询结果缓存到缓存中。下次相同查询直接从缓存取。事务提交/关闭时清空。作用域小，适合短生命周期操作。
- **二级缓存**：Mapper级别（需手动开启，在mapper.xml配置`<cache>`）。跨SqlSession共享，存储在 PerpetualCache（默认HashMap，可用Ehcache/Redis替换）。查询先查二级、再一级、再数据库。更新操作会清空相关缓存。支持序列化，适合读多写少场景。

注意：一级缓存线程不安全，多线程需新SqlSession；二级缓存需配置eviction策略（如LRU）避免内存溢出。

>  [!important]
>
> 类似：
>
> ​	隔离级别：可重复读
>
> 可能会问：
>
> 1. `#`和`$`的区别
> 2. 问到底层原理则战略性放弃