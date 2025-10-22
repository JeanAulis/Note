[toc]

---

## 1. Final有什么用

- 对于类来说，被final修饰的类不可以被继承
- 对于方法来说，被final修饰的方法不可以被重写
- 对于变量来说，被final修饰的变量不可以被改变
- final修饰不可变的是变量的引用，而不是引用指向的内容，**引用指向的内容可以改变**



## 2. 什么是重载（Overload）和重写（Override） ?

- **重载**是发生在同一个类里面，必须方法名相同，方法的参数列表必须不同，也就是参数类型和个数还有顺序至少有一个不同，重载是通过方法参数来区分的，而不是通过返回类型。
- **重写**则发生在父子类中，方法名，参数列表必须相同。返回值、抛出的异常，小于等于父类，访问修饰符 private < default < protect < public 必须大于等于父类，当父类为`private`则不能重写

<font style="color:red">// TODO private < default < protect < public </font>




## 3. 重载的方法能否根据返回类型进行区分？

方法重载不可以根据返回类型区分



## 4. ==和equals的区别？equals相等hashcode一定相等吗？hashcode相等equals一定相等吗？什么场景需要重写equals和hashcode方法？具体怎么重写equals和hashcode方法？

- == 是判断两个对象的地址是不是相等，就是判断两个对象是不是同一个对象。对于基本数据类型：==比较的是值，应用数据类型 ： == 比较的是内存地址。
- equals()：和==相同，比较的是内存地址，但是equals()可以被重写，用来比较对象内部的属性值，对于字符串和包装类，使用equals()可以避免缓存陷阱

一定。果两个对象根据 equals 比较相等，那么它们的 hashCode 必须相同；否则会导致哈希容器（例如 HashMap、HashSet）中无法正确查找、删除或存取数据。

不一定，因为存在哈希碰撞

主要是在对象需要放入基于哈希结构的集合（例如 HashMap 的 key、HashSet 的元素）时。如果不重写，就会导致集合中判断重复、查找或删除操作不准确。例如，如果两个业务对象代表相同的用户，但没重写 equals 和 hashCode，集合就可能把它们当成两个不同的元素。

#### 重写equals和hashCode()





## 5. String和StringBuffer、StringBuilder的区别是什么？

- String具有不可变性：底层通过一个 final 的字符数组保存内容（JDK 9 开始 String 的底层是 byte[] 而不是 char[]），所以String对象是不可变的，StringBuilder与StringBuffer这两种对象是可变的（底层实现的是动态扩容的char[]）
- 线程安全：String因为其不可变，天然线程安全。StringBuffer对方法加了同步锁`synchronized`或者对调用的方法加了同步锁，所以是线程安全的。StringBuilder并没有对方法进行加同步锁，所以是非线程安全的。
- 性能：每次对String类型进行改变的时候，都会生成一个新的String对象，然后将指针指向新的String对象，性能较差。StringBuffer每次都会对StringBuffer对象本身进行操作，而不是生成新的对象并改变对象引用。StringBuilder相比使用StringBuffer而言效率更高。

> StringBuilder（extends AbstractStringBuilder）中几乎所有的修改方法返回值都是this，这样就可以链式调用。**流式（Fluent）接口设计思想**
>
> StringBuffer也是（extends AbstractStringBuilder），和StringBuilder一致，只是所有方法都加了`synchronized`
>
>  **建造者模式 (Builder Pattern)**



## 6. java中基本类型有哪些？

| 基本类型  | 包装类      | 描述                                                | 占用字节数 |
| :-------- | :---------- | --------------------------------------------------- | :--------- |
| `byte`    | `Byte`      | 8 位有符号整数                                      | 1          |
| `short`   | `Short`     | 32 位有符号整数                                     | 2          |
| `int`     | `Integer`   | 64 位有符号整数                                     | 4          |
| `long`    | `Long`      | 32 位单精度浮点数 (IEEE 754)                        | 8          |
| `float`   | `Float`     | 64 位双精度浮点数 (IEEE 754)                        | 4          |
| `double`  | `Double`    | 64 位双精度浮点数 (IEEE 754)                        | 8          |
| `char`    | `Character` | 16 位 Unicode 字符                                  | 2          |
| `boolean` | `Boolean`   | 逻辑值，只有 $\text{true}$ 和 $\text{false}$ 两个值 | 1          |

> 对于boolean占用字节数：虚拟机规范中没有明确规定，但通常被认为占用 1 个字节（或一个 $\text{int}$ 的空间用于数组中的存储）

## 7. 什么是反射？

反射就是允许程序在运行过程中动态地获取类的信息（成员变量、方法、构造器等）。

>  反射（Reflection）是Java中一种运行时动态获取类信息和操作类或对象的功能。通过反射，可以获取类的结构（如方法、字段、构造器），动态调用方法或修改字段值。常用于框架开发（如Spring），但性能开销较高，需谨慎使用。

>  [!tip]
>
>  获取class的方法
>
>  在Java中，获取`Class`对象的方法主要有以下三种：
>
>  1. **通过类名**：`Class<?> clazz = ClassName.class;`  
>     直接使用类的字面量，适用于已知类名的情况。
>
>  2. **通过对象实例**：`Class<?> clazz = instance.getClass();`  
>     从对象实例获取其对应的`Class`对象。
>
>  3. **通过全限定类名**：`Class<?> clazz = Class.forName("package.ClassName");`  
>     动态加载类，需提供完整类路径，适用于运行时加载未知类。
>
>  注意：`Class.forName()`可能抛出`ClassNotFoundException`，需处理异常。



## 8. 反射机制优缺点

优点： 运行期类型的判断，动态加载类，提高代码灵活度。

缺点： 性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，性能比直接的java代码要 慢很多





## 9. (fuck)在你进行项目开发的过程中有没有用到过反射 

在我们的项目中经常会使用反射 + 自定义注解的方式去实现一些功能 , 例如 : 

1. 在前后端交互的时候, 后端Long类型返回前端后会产生精度丢失 , 我们的处理方式就是在服务端, 通过配置修改Jackson的序列化规则, 将一些Long类型字段转化为字符串返回给前端, 这个时候我们自定义了一个@IdEncrpt注解 , 通过反射获取类的属性, 判断属性上是否添加了@IdEncrpt注解, 如果添加了 , 就会通过反射获取属性值, 转化为字符串
2. 在整合EMQ的时候 , 为了能够方便的接收订阅消息, 我们自定义了一个@Topic注解 , 作用在类上 , 之后我们通过反射获取类的字节码, 并且获取类上的@Topic注解, 读取到里面定义的主题 , 通过策略模式将不同主题的消息分发到不同的处理器中
3. 除了上述之外, 在我们项目开发中经常使用的一些框架, 例如 : Mybatis , Spring , SpringMVC 等, 以及一些常用的工具库 common-utils , hutool工具库等都大量使用到了反射机制

<font style="color:blue">// TODO</font>



## 10. springboot模式是单例还是多例？有哪些作用域？我怎么修改为多例？

- 单例。

> `@Component`、`@Service`、`@Repository`等注解

- 作用域有五种常用的有`singeton`和`prototype`

> <font style="color:red">// TODO 其他</font>

- 如何使用？=> 使用`@Autowired`注解或者`ApplicationContext.getBean()`获取Bean
- 要修改为多例，需要在@Component或@Bean上添加@Scope("prototype")注释
- 在@Configuration类中定义Bean时指定作用域。
- 动态获取prototype Bean：使用ApplicationContext或ObjectFactory获取新实例。



## 11. Integer i1 = 127;Integer i2 = 127;i1==i2？Integer i3 = 128;Integer i4 = 128;i3==i4？int a = 128;Integer b = 128;a==b？Integer i = 128和new一个有什么区别？

== 是判断两个对象的地址是不是相等，就是判断两个对象是不是同一个对象。对于基本数据类型：==比较的是值，应用数据类型 ： == 比较的是内存地址。

true，缓存复用：因为Integer会在缓存（Integer Cache）中保存-128~127的对象

false，超过了缓存范围，创建不同对象

相等。拆箱比较值：基本数据类型int和Integer用`==`比较时，Integer会自动拆箱为int，然后进行数值比较。

>  [!tip]
>
>  建议使用`.equals()`进行比较`Integer`而不是`==`

`Integer i = 128`会自动装箱，超出缓存范围的会创建新对象，而`new Integer(128)`总是创建新的对象，不使用缓存机制



## 12. 接口和抽象类的区别

首先，从定义和实现的角度来说，接口在 Java 8 之前只能定义抽象方法；从 Java 8 开始可以包含默认方法和静态方法，从 Java 9 起支持私有方法，用于默认方法的内部复用。
实现类必须提供所有方法的实现**Java 8** 开始允许在接口中定义 **默认实现方法**，用 default 关键字修饰，但是**Object类方法不能定义为default**；抽象类可以包含抽象方法和具体方法，子类需要实现抽象方法，可以继承具体方法。

> 原因是所有类都隐式继承 Object，如果接口里定义了和 Object 同名的默认方法，会造成二义性（比如 toString、equals），所以 Java 规范禁止这么做。

从多来说继承性方面来说，接口支持多实现，一个类可以实现多个接口；抽象类不支持多继承，抽象类也是单继承。

还有接口只能定义常量，而且没有构造方法；抽象类可以定义普通成员变量和常量，也有构造方法，子类实例化的时候会调用。

**适用场景**：接口适合指定某一种标准或者规范，抽象类适合适合定义有继承关系的类层次结构



## 13. 说一下java的三大特性

> 封装、继承、多态

封装: 封装是将数据（属性）和操作数据的方法（行为）结合在一起，并对外隐藏对象的内部实现细节。通过访问修饰符（public、private、protected）来控制对类成员的访问权限。

继承: 继承允许一个类（子类/派生类）获得另一个类（父类/基类）的属性和方法，实现代码重用。Java使用`extends`关键字实现继承。

多态: 多态是指同一个接口或方法在不同的对象上有不同的实现方式。包括方法重载（编译时多态）和方法重写（运行时多态）。



## 14. 深拷贝和浅拷贝的区别？怎么实现深拷贝？

> **Java 是值传递**

浅拷贝和深拷贝是对象复制的两种方式，区别在于处理引用类型属性时。

- **浅拷贝**：只复制基本类型（如int、String）的值，对于引用类型（如对象、数组），复制的是引用地址。新旧对象共享同一内存，修改一个会影响另一个。适合简单对象，但容易导致意外修改。

> Java中Object.clone()（native方法）默认是浅拷贝（需实现Cloneable接口，否则会抛出 CloneNotSupportedException）。

- **深拷贝**：复制整个对象图，包括所有引用类型的内部对象。新对象完全独立，修改不影响源对象。适合复杂嵌套对象，如项目中的用户对象（含设备列表）。数组也有深拷贝，不需要实现Cloneable接口。

> [!tip]
>
> ```Java
> int a = 1;
> int b = a;
> b = 2;
> System.out.println(a); // 输出1
> ```
>
> 浅拷贝是对于引用类型而言的，即拷贝的是对象的引用，从而两个变量指向同一个对象。而基本数据类型不存在引用，直接就是值的拷贝。
>
> 从JVM角度考虑为什么是1<font style="color:red">// TODO</font>

### 如何实现深拷贝？

> [!tip] 
>
> 深度复制的方式：
>
> 1. **手动深度复制**：通过手动复制每个属性，确保引用类型的属性是通过新对象来复制。
> 2. **序列化**和**反序列化**：使用 `ByteArrayOutputStream` 和 `ObjectInputStream` 将对象转化为字节流，再反序列化回新对象，从而实现深度复制。性能差。
> 3. ~~**工具库**：使用 Apache Commons Lang 提供的 `SerializationUtils` 来简化深度复制的实现。~~

- [x] 会不会触发构建方法

**不会**





## 15. stream流你熟悉吗？说一下常用的api？

熟悉。常用的一些api有map()对一些对象进行映射，collect()收集Stream流，filter对一些元素进行过滤，sorted()排序，distinct()去重，forEach()遍历循环元素，limit()取前几个元素，skip()跳过前几个元素，count()统计个数，还有最大值最小值max()、min()。大概这些常用的api。

- ~~stream()：从集合创建流。~~

- map()：对一些对象的属性做映射
- filter()：按条件过滤元素
- distinct()：去重（依赖 `hashCode` 和 `equals`  => 引出重写`hashCode`和`equals`方法）
- limit(n)：截取前n个
- skip(n)：跳过前n个
- collect()：收集stream流
- forEach()：循环遍历元素
- sorted()：排序
- count()、max()、min()



## 16. java常见的集合类有哪些

Map接口和Collection接口是所有集合框架的父接口： 

1. Collection接口的子接口包括：Set接口和List接口 
2. Set接口的实现类主要有：HashSet、TreeSet、LinkedHashSet等 
3. List接口的实现类主要有：ArrayList、LinkedList、~~Stack以及Vector~~等
4. Map接口的实现类主要有：HashMap、TreeMap、Hashtable、ConcurrentHashMap以及 Properties等 



## 17. 什么是 Java 泛型

- 泛型的主要目的是实现**类型参数化**，java 在定义类、定义接口、定义方法时都支持泛型

泛型的好处有

- 提供编译时类型检查，避免运行时类型转换错误，提高代码健壮性
- 设计更通用的类型，提高代码通用性



## 18. Object的常用Api有哪些？

- equals(Object obj)：判断对象内容是否相等（默认实现是 ==，可重写实现按内容比较）。
- hashCode()：返回对象的哈希码，通常与 equals 配合重写，用于哈希结构（HashMap、HashSet）中。
- getClass()：获取对象的运行时类类型。
- toString()：返回对象的字符串表示，默认返回 "类名@哈希码"，通常重写以输出有意义的信息。
- clone()：创建对象的浅拷贝，需要实现 Cloneable 接口。
- wait() / wait(long timeout) / wait(long timeout, int nanos)：使当前线程等待，必须在同步块内调用。
- notify()：唤醒一个正在等待该对象监视器的线程。
- notifyAll()：唤醒所有等待该对象监视器的线程。
