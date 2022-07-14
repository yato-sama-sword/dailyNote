# Spring

## Spring MVC

### 什么是MVC

> model-view-controller，是一种软件设计规范，本质上是一种解耦，其中：
> 
> 1. Model是应用程序中用于处理应用程序数据逻辑的部分（JavaBean对象
> 2. View是应用程序中处理数据显示的部分，一般需要依据模型数据创建（Html页面
> 3. Controller是应用程序中处理用户交互的部分，通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据（emmm，就是Controller或者说Servlet

### 什么是Spring MVC

> 是遵从MVC规范开发出的Web框架

### Spring MVC请求流程

> **注！！！用了kaitao-springMVC书中的知识（或者说博客？**
> 
> DispatcherServlket相当于前端控制器，Handler相当于页面控制器；前者负责发送具体请求给页面控制器，后者则负责处理对应的业务逻辑，实现功能，完成功能需要将ModelAndView还回去。最后进行视图的解析，前端控制器会对用户做出响应

1. **首先用户发送请求——>DispatcherServlet**，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；
2. **DispatcherServlet——>HandlerMapping**， HandlerMapping 将会把请求映射为 HandlerExecutionChain 对象（包含一个 Handler 处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象，通过这种策略模式，很容易添加新的映射策略；
3. **DispatcherServlet——>HandlerAdapter**，HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器， 即适配器设计模式的应用，从而很容易支持很多类型的处理器；
4. **HandlerAdapter——>处理器功能处理方法的调用**，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处 理方法，完成功能处理；并返回一个ModelAndView 对象（包含模型数据、逻辑视图名）；
5. **ModelAndView 的逻辑视图名——> ViewResolver**，ViewResolver 将把逻辑视图名解析为具体的View，通过这种策 略模式，很容易更换其他视图技术；
6. **View——>渲染**，View 会根据传进来的Model 模型数据进行渲染，此处的Model 实际是一个Map 数据结构，因此 很容易支持其他视图技术；
7. **返回控制权给DispatcherServlet**，由DispatcherServlet 返回响应给用户，到此一个流程结束。

## Spring Framework

> **个人认为核心是IOC容器、AOP代理，需要指出SpringBoot对约定优于配置的实践最佳**

### 框架

> 脚本之家版：
> 
> 1. **Core模块**，实现IOC，提供框架基础功能。BeanFactory类是Spring核心类，负责JavaBean的配置与管理。采用Factory模式实现了IOC即依赖注入。JavaBean是一种Java语言写成的可宠用组件，必须是具体类和公共类，并且有无参数的构造器
> 2. **Context模块**，继承BeanFactory类，并且添加事件处理、国际化等等功能
> 3. **AOP模块**，实现Spring管理对象AOP化
> 4. **DAO模块**，将业务逻辑代码与数据库交互代码分离
> 5. **ORM模块**，提供对现有ORM框架支持，指出有一个叫Spring Data JPA的东西，这个也是搞ORM的，但是mybatis做动态sql更有优势
> 6. **Web模块**，建立在Context模块基础上，兼容现有Web框架，并提供Servelet监听器的Context和Web应用上下文。并且推荐使用thymeleaf模板。
> 7. **MVC模块**，建立在Core模块基础上，引入控制器的概念：事务逻辑是Model，界面是View，Controller是根据请求调用事务逻辑。

> JavaGuide版：
> 
> ![Spring主要模块](file://C:\Users\yato\Desktop\U know wt\面试准备\dailyNote\笔记.assets\e0c60b4606711fc4a0b6faf03230247a.png?msec=1657797480991)

### 工作原理

让模块和模块之间不通过代码关联，而是通过Spring配置进行反射来动态的组装

#### IOC（控制反转）:

他妈的甚么叫控制反转？

1. **控制**：对象创建（实例化、管理）的权力
2. **反转**：控制权交给外部环境（Spring框架、IOC容器）

啊，他妈的，让原本在程序中手动创建对象的控制权交给Spring框架，这就叫控制反转。

具体来说IoC容器实际上是一种Map，存放各种对象。在SpringBoot流行之前常常采用XML文件来配置，如今改用注解配置

#### AOP（面向切面编程）：

将一些与业务无关，却为业务模块所共同调用的逻辑或责任（日志管理、权限控制等）封装起来，减少系统的重复代码，降低模块间的耦合度，有利于未来的可拓展性和可维护性，可以基于动态代理实现。

> **神奇的动态代理：**
> 
> 1. JDK Proxy：只能够代理实现了接口的对象
> 2. Cglib：甚么都可以代理嘞！
> 
> ![SpringAOPProcess](file://C:\Users\yato\Desktop\U know wt\面试准备\dailyNote\笔记.assets\926dfc549b06d280a37397f9fd49bf9d.jpg?msec=1657797480991)

以上范围属于Spring AOP，至于AspectJ AOP，相较于前者它属于编译时增强，基于字节码操作。不仅更强大，而且更简单。如果切面太多，可以选择AspectJ，快的很！

### Bean

代指被IoC容器所管理的对象，通过配置元数据定义。配置元数据可以是XML文件、注解或者Java配置类。可以通过注解或者xml设置bean的作用域（有singleton、prototype、request、session、global-session）

声明bean的注解有：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。

#### bean的生命周期：

总的来说：分为实例化、属性设置、初始化、销毁四个阶段。具体如下图所示：

![img](file://C:\Users\yato\Desktop\U know wt\面试准备\dailyNote\笔记.assets\Bean生命周期-16519280909622.png?msec=1657797481022)

### 事务

#### 管理方式

1. **编程式事务管理**：通过 `TransactionTemplate`或者`TransactionManager`手动管理事务，实际应用中很少使用。
2. **声明式事务管理**：基于AOP实现（代码侵入性小，可以通过@Transactional注解实现）

> 额外指出，一般来说事务的回滚是在遇到神奇的运行时错误时发生。如果设置@Transactional(rollbackFor = Exception.class)，那么在遇到非运行时异常时也会回滚(**不是Error，请注意**)

#### 管理接口

1. **`PlatformTransactionManager`**： （平台）事务管理器，Spring 事务策略的核心。
2. **`TransactionDefinition`**： 事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)。
3. **`TransactionStatus`**： 事务运行状态。

我们可以把 **`PlatformTransactionManager`** 接口可以被看作是事务上层的管理者，而 **`TransactionDefinition`** 和 **`TransactionStatus`** 这两个接口可以看作是事务的描述。

**`PlatformTransactionManager`** 会根据 **`TransactionDefinition`** 的定义比如事务超时时间、隔离级别、传播行为等来进行事务管理 ，而 **`TransactionStatus`** 接口则提供了一些方法来获取事务相应的状态比如是否新事务、是否可以回滚等等。

#### 传播行为

**事务传播行为是为了解决业务层方法之间互相调用的事务问题**。

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

正确的事务传播行为可能的值如下:

1. **`TransactionDefinition.PROPAGATION_REQUIRED`**
   
   ​ 使用的最多的一个事务传播行为，我们平时经常使用的`@Transactional`注解默认使用就是这个事务传播行为。如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务

2. **`TransactionDefinition.PROPAGATION_REQUIRES_NEW`**
   
   ​ 创建一个新的事务，如果当前存在事务，则把当前事务挂起。也就是说不管外部方法是否开启事务，`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。

3. **`TransactionDefinition.PROPAGATION_NESTED`**
   
   ​ 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`TransactionDefinition.PROPAGATION_REQUIRED`。

4. **`TransactionDefinition.PROPAGATION_MANDATORY`**
   
   如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性），实际使用的很少。

若是错误的配置以下 3 种事务传播行为，事务将不会发生回滚：

- **`TransactionDefinition.PROPAGATION_SUPPORTS`**: 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`**: 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **`TransactionDefinition.PROPAGATION_NEVER`**: 以非事务方式运行，如果当前存在事务，则抛出异常。

### 设计模式

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。

### Spring MVC原理

1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler`来调用真正的处理器开处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）

### JPA

Java持久层API的意思，是ORM框架的规范。如果想要一个字段不被数据库持久化，可以采用以下的方法：

```java
static String transient1; // not persistent because of static
final String transient2 = "Satish"; // not persistent because of final
transient String transient3; // not persistent because of transient
@Transient
String transient4; // not persistent because of @Transient
```

## Spring Boot

### 优势

1. 自动配置：针对很多Spring应用程序常见的应用功能，Spring Boot能自动提供相关配置
2. 起步依赖：告诉Spring Boot需要什么功能，它就能引入需要的库。
3. 命令行界面：这是Spring Boot的可选特性，借此你只需写代码就能完成完整的应用程序，无需传统项目构建。
4. Actuator：让你能够深入运行中的Spring Boot应用程序，一探究竟。

### 自动装配原理

核心是通过`@EnableAutoConfiguration`这个注解来调用**AutoConfigurationImportSelector**类实现自动装配。而这个类中的重点方法为：`getAutoConfigurationEntry`，首先判断自动装配功能是否打开，然后获取注解中的**exclude和excludeName**，最终读取`META-INF/spring.factories`文件来获取需要自动装配的所有配置类。（注意：不仅会读取当前依赖下的，而是读取所有Spring Boot Starter下的

ps：spring.factories中的配置很多，不是每次启动都需要全部加载，需要满足`@ConditonalOnXXX`注解中所设置的条件

### 常用注解

注解的具体学习可以看别人的项目是怎么用的

### 1.`@SpringBootApplication`

可以`@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合

### 2.Bean相关

我们一般使用 `@Autowired` 注解让 Spring 容器帮我们自动装配 bean。要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,可以采用以下注解实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

对于这些Bean，我们可以通过`@Scope`来声明其作用域，接下来展示下**四种常见的 Spring Bean 的作用域：**

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
- session : 每一个 HTTP Session 会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。

`@RestController`注解是`@Controller`和`@ResponseBody`的合集,表示这是个控制器 bean,并且是将函数的返回值直接填入 HTTP 响应体中,是 REST 风格的控制器。这标志着类中的所有方法返回的都是JSON格式的数据（**所以是controller里的！**

`@Configuration`一般用来声明配置类，表示该类是一个config，可以用`Component`来代替，但是就用这个更好嘞

`@Bean`可以用在方法上，表示这个方法将生成一个bean对象，交给Spring容器管理

### 3.处理常见的HTTP请求类型

- **GET** ：请求从服务器获取特定资源。举个例子：`GET /users`（获取所有学生）
- **POST** ：在服务器上创建一个新的资源。举个例子：`POST /users`（创建学生）
- **PUT** ：更新服务器上的资源（客户端提供更新后的整个资源）。举个例子：`PUT /users/12`（更新编号为 12 的学生）
- **DELETE** ：从服务器删除特定的资源。举个例子：`DELETE /users/12`（删除编号为 12 的学生）
- **PATCH** ：更新服务器上的资源（客户端提供更改的属性，可以看做作是部分更新），使用的比较少，这里就不举例子了。

注意，根据Restful规则，URL中是不能出现动词的，可以用动词对应的名词代替，这表示一种服务而非动作

对于这些请求的注解：通用的是`@RequestMapping`，其value属性对应请求的url地址，其method属性对应请求的类型。当然也可以直接用`@GetMapping`、`@PostMapping`这一类的注解。

### 4.前后端传值

上面提到了请求类型，咱这一块请求数据必须得先跟上

`@PathVariable`可以获取路径中的参数，`@RequestParam`可以用于获取查询的参数（换言之就是请求得到的数据，`@RequestBody`注解则直接读取Request请求中的body部分，并且将application/json格式的数据直接转换为Java对象！由于使用该注解已经获取了所有的body信息，如果需要进行多次注解，一般来说是G了

### 5.读取配置信息

这里只说常用的啊，`@value`可以从配置文件中读取特定的配置信息，`@ConfigurationProperties`可以读取配置信息，并且与bean进行绑定。这个听起来比较抽象，举个例子，加入我们的application.yml长这样

```yaml
wuhan2020: 2020年初武汉爆发了新型冠状病毒，疫情严重，但是，我相信一切都会过去！武汉加油！中国加油！

my-profile:
  name: Guide哥
  email: koushuangbwcx@163.com

library:
  location: 湖北武汉加油中国加油
  books:
    - name: 天才基本法
      description: 二十二岁的林朝夕在父亲确诊阿尔茨海默病这天，得知自己暗恋多年的校园男神裴之即将出国深造的消息——对方考取的学校，恰是父亲当年为她放弃的那所。
    - name: 时间的秩序
      description: 为什么我们记得过去，而非未来？时间“流逝”意味着什么？是我们存在于时间之内，还是时间存在于我们之中？卡洛·罗韦利用诗意的文字，邀请我们思考这一亘古难题——时间的本质。
    - name: 了不起的我
      description: 如何养成一个新习惯？如何让心智变得更成熟？如何拥有高质量的关系？ 如何走出人生的艰难时刻？
```

那么这两个注解可以长这样

```java
@Value("${wuhan2020}")
String wuhan2020;
```

```java
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
    @NotEmpty
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
  省略getter/setter
  ......
}
```

### 6.参数校验

这里引出JSR参数校验的概念，这是一套JavaBean参数校验的标准。**由于前端校验可能存在的一些问题，依然需要对传入后端的数据进行一次校验，避免用户绕过浏览器直接通过一些HTTP工具直接向后端请求一些违法数据嘞**

- `@NotEmpty` 被注释的字符串的不能为 null 也不能为空
- `@NotBlank` 被注释的字符串非 null，并且必须包含一个非空白字符
- `@Null` 被注释的元素必须为 null
- `@NotNull` 被注释的元素必须不为 null
- `@AssertTrue` 被注释的元素必须为 true
- `@AssertFalse` 被注释的元素必须为 false
- `@Pattern(regex=,flag=)`被注释的元素必须符合指定的正则表达式
- `@Email` 被注释的元素必须是 Email 格式。
- `@Min(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@Max(value)`被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@DecimalMin(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@DecimalMax(value)` 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@Size(max=, min=)`被注释的元素的大小必须在指定的范围内
- `@Digits(integer, fraction)`被注释的元素必须是一个数字，其值必须在可接受的范围内
- `@Past`被注释的元素必须是一个过去的日期
- `@Future` 被注释的元素必须是一个将来的日期
- ......

验证请求体和验证请求参数有一丢丢小小的区别，如果需要验证参数的话，需要额外**在类上加上 `@Validated` 注解**

### 7.异常处理

1. `@ControllerAdvice` :注解定义全局异常处理类
2. `@ExceptionHandler` :注解声明异常处理方法

一般来说我们处理的是RuntimeException，所以我们的自定义异常类型可以直接继承这个类。对于异常的信息，我们可以封装好返回给前端，这样对用户更加友好。想要一次性处理多种异常还可以使用枚举，比较方便。这里举一个例子

```java
@ControllerAdvice(assignableTypes = {ExceptionController.class})
@ResponseBody
public class GlobalExceptionHandler {
    ErrorResponse illegalArgumentResponse = new ErrorResponse(new IllegalArgumentException("参数错误!"));
    ErrorResponse resourseNotFoundResponse = new ErrorResponse(new ResourceNotFoundException("Sorry, the resourse not found!"));
    @ExceptionHandler(value = Exception.class)// 拦截所有异常, 这里只是为了演示，一般情况下一个方法特定处理一种异常            public ResponseEntity<ErrorResponse> exceptionHandler(Exception e) {
        if (e instanceof IllegalArgumentException) {    
            return ResponseEntity.status(400).body(illegalArgumentResponse);
        } else if (e instanceof ResourceNotFoundException) {
            return ResponseEntity.status(404).body(resourseNotFoundResponse);
        }     
        return null;
    }
}
```

### 8.JPA相关

`@Entity`声明一个类对应一个数据库实体。

`@Table` 设置表名

```java
@Entity
@Table(name = "role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    省略getter/setter......
}
```

## Spring Cloud

> 快速入门分布式，还得看我Spring Cloud 大将军嘞。构建微服务的一整套流程一般有**服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控**等操作！使用Spring Cloud就可以在Spring Boot项目基础上轻松实现微服务项目搭建

### Eureka

> Spring Cloud中的服务发现框架

首先简单讲讲什么是服务发现，在服务发现中有三个角色。**服务提供者、服务消费者、服务中介**，顾名思义，分别负责提供服务、使用服务，构建使用服务与提供服务的中间桥梁。这个桥梁要怎么当？这里两个核心概念**服务注册**：Eureka客户端（这里是服务提供者）向Eureka Server 注册时，需要自身的元数据（**ip地址、端口、运行状况之类的**）；**获取注册列表信息**：Eureka客户端（这里是服务消费者）从服务器获取注册表信息，将其缓存在本地。客户端可以使用该信息查找其他服务，从而进行远程调用。这个缓存在本地的注册表信息会定期与服务器同步更新嘞。哦，还有一个很关键的概念是**服务续约**，就是**服务提供者**定时向**服务中介（Eureka Server）**发送心跳进行续约(通俗一点就是告诉服务中介我还在提供服务，定时的告诉)

还有两个概念也很重要，一个是**服务下线**，一个是**服务剔除**，前者是服务提供者主动向服务中介告知，不再提供服务；后者是服务提供者多次不进行服务续约，服务中介认为服务提供者不再提供服务了，就下线了该服务

Eureka的自我保护机制也简单介绍一下，就是网络不好或者有啥外界条件影响时，就算长期没有进行服务续约，也不会选择下线服务，因为Eureka可能认为是网的原因，也许服务提供者是发送了心跳的，但是自己没收到

### Ribbon

> Spring Cloud中的负载均衡

先讲讲`RestTemplate`，作为Spring提供的一个访问Http服务的客户端类，在微服务之间进行调用时候会常常使用

Ribbon是一个**运行在消费者端的**，客户端/进程内负载均衡器，可以在Consuker端获取到所有服务列表后，在内部使用负载均衡算法，实现对多个系统的调用。这里不妨提一下Nignx（**集中式的负载均衡器**）。这个说法应该很清楚的指出了二者的区别，在Nginx中请求会先进入负载均衡器，而在Ribbon中是再客户端进行负载均衡后发送请求。

默认采用的负载均衡算法是**轮询**：若经过一轮轮询没有找到可用的Provider，其最多轮询 10 轮。若最终还没有找到，则返回。

### Open Feign

> 使用HttpClient相关技术调用接口，说到接口其实得提一嘴Swagger，后者是编写接口的。

服务代码映射到消费者端，底层协议采用HTTP，并且使用Rest API。使用@FeignClient注解可以将类注册到容器中，通过动态代理创建被调用类实例，在Controller内就可以随心调用了

### Hystrix

> 熔断+降级，系统容错有保障

先讲讲服务雪崩的概念，一个服务崩溃了，导致其他依赖于该服务的服务一起崩溃。**熔断**指的就是当一定时间窗口内的请求失败率到达设定阈值时，系统通过**断路器**直接将此请求链路断开，对应Hystrix中的断路器模式。**降级**相比之前粗暴的不提供服务了，换了一种温柔的方式，比方说调用另一个方法提示消费者等会再来消费，更加友好，这u敌营Hystrix中的后备处理模式

### Zuul

> 是从前端到后端的一道大门！可以实现动态路由、监视、弹性和安全性，概念很高级。但是从网关概念出发，就是对请求进行**鉴权、限流、路由、监控**

#### 路由

Zuul可以在Eureka注册，并获取所有Consumer信息，并直接进行路由映射。使用只需要注册Eureka，并且在启动类上加入@EnableZuulProxy注解。此外还可以实现统一前缀（路径统一加一个prefix），路由策略配置（自定义路径替代微服务名称）、服务名屏蔽、路径屏蔽等等

#### 过滤

可以自定义Filter继承ZuulFilter并注入Spring容器，可以轻松实现请求时间日志打印、令牌桶限流等等功能

### Config + Spring Cloud Bus

> 使用Config服务器，可以在中心位置管理所有环境中应用程序的外部属性，听起来抽象，其实就是可以去git实时拉去配置

> 将服务和服务示例和分布式消息系统连接在一起，在集群中传播状态更改。有点像广播站哈

使用Spring Cloud Bus + Config，在请求上加上@ResfreshScope注解可以进行配置的动态修改（直接从Git远程配置库拉取请求，实时更新）