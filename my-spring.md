# 项目前置介绍

>Spring的目的在于使JAVA EE开发更加容易，个人认为其核心在于IOC以及AOP。IOC是一种理念，将对象的创建权交给Spring，而AOP是一个模块，实现面向切面编程，比较接近一种动态增强。个人将着重IOC、AOP的相关功能进行实现。

## Spring FrameWork组件

>其实组件不止下述这些，没用到的没有提及。想快速了解可以看看[Spring FrameWork 5](https://pdai.tech/md/spring/spring-x-framework-introduce.html)专栏，鱼皮大大写的。想深入了解的话推荐看陈雄华老师在电子工业出版社出版的Spring有关书籍，讲的很细，日后希望可以详读。

### Core Container（Spring核心容器）

> 实现其他模块的基础，由Beans模块、Core核心模块、Context上下文模块和SpEL表达式语言模块组成。想在项目中实现AOP等功能必须仰赖于该模块。下面介绍一下SpEL模块外的三个模块

- **Beans 模块**：提供了框架的基础部分，包括控制反转和依赖注入。
- **Core 核心模块**：封装了 Spring 框架的底层部分，包括资源访问、类型转换及一些常用工具类。
- **Context 上下文模块**：建立在 Core 和 Beans 模块的基础之上，集成 Beans 模块功能并添加资源绑定、数据验证、国际化、Java EE 支持、容器生命周期、事件传播等。ApplicationContext 接口是上下文模块的焦点。

### AOP

> 提供了面向切面编程实现，提供比如日志记录、权限控制、性能统计等通用功能和业务逻辑分离的技术，并且能动态的把这些功能添加到需要的代码中，这样各司其职，降低业务逻辑和通用功能的耦合。

### Aspects

> 提供与 AspectJ 的集成，是一个功能强大且成熟的面向切面编程（AOP）框架。（和Spring AOP实现上没有关系，只是Spring可以集成AspectJ，使用相关方法）

## IOC

>控制反转，反转的是依赖对象的获取。由容器创建对象并且进行管理（在合适的时候注入），取代了本来由我们自己创建获取对象的操作。在Spring Boot中使用@Autowired注解就搞定对象注入，在项目中得深入看看背后的过程了。

### 配置方式

1. xml配置：挺麻烦的，创建xml文件声明命名空间和配置bean，bean多的话能配十万八千行，但是适用于任何场景
2. Java配置：声明一个Config类，注解`@Configuration`，在方法中注解@Bean创建对应实例并返回。同样适用于任何场景，但是大量配置可读性就比较差
3. **注解配置：**简单易用，打个注解就行，但是较之于前两种方法，不能适用于所有场景，通常情况下采用这种方法

### 依赖注入方式

1. 构造方法注入：正如其名，在调用构造函数时，实现依赖对象的注入
2. setter注入：正如其名，使用set方法实现依赖对象注入
3. 注解注入：使用`@Autowired`注解进行注入

官方推荐构造方法注入，既保证注入组件不可变，又确保需要的依赖不为空，这里就得看下具体的注入方法了

```java
@Service

public class ServiceImpl {
    
    private final DaoImpl daoImpl;
    
    public ServiceImpl (final DaoImpl daoImpl) {
		this.daoImpl = daoImpl;
	}
}

```

1. final保证依赖对象是不可变的
2. 由于实现有参构造函数，就必须传入对应参数才能构造对象
3. 可以获取原始对象，即构造方法创建出来的对象
4. 可以解决循环依赖问题，采用构造方法注入时。spring项目启动会抛出：BeanCurrentlyInCreationException：Requested bean is currently in creation: Is there an unresolvable circular reference？从而提醒你避免循环依赖

### 整体功能

>总体来说功能如下图所示嘞：

![IOC功能](\笔记.assets\spring-framework-ioc-source-7.png)

1. 加载Bean配置：对不同类型资源的加载（比如xml配置文件），解析成生成统一Bean的配置（Bean容器，对Bean进行定义和行为管理）
2. 根据Bean定义加载生成Bean实例，并放置在Bean容器中：比如Bean依赖注入、Bean嵌套、Bean缓存
3. 特殊Bean的处理：比如国际化Message等生成特殊类结构支撑
4. 对容器中Bean提供统一管理和调用：一般来说采用工厂模式

### 体系结构

>主要分为三大块！：
>
>1. BeanFactory和BeanRegistry：定义IOC容器功能规范和Bean的注册
>2. BeanDefinition：定义各种Bean对象及其相互的关系
>3. ApplicationContext：IOC容器的接口类，在基本IoC容器实现基础上实现访问资源、国际化、应用事件等功能

#### BeanFactory

>列出实现的方法如下：

1. 根据bean名字和Class类型来获得bean实例
2. 返回指定bean的Provider
3. 检查工厂中是否包含给定name的bean，或者外部注册的bean
4. 检查所给定name的bean是否为单例/或原型
5. 判断所给name的类型与type是否匹配
6. 获取给定name的bean的类型
7. 返回给定name的bean的别名（Aliases）

#### BeanRegistry

>Spring配置文件中的每一个`bean`在Spring容器中都会通过BeanDefinition对象表示。BeanDefinitionRegistry接口提供向容器手工注册BeanDefinition对象的方法

#### BeanDefinition

>对Bean对象和对象间关系进行定义，相关的还有BeanDefinitionReader（BeanDefinition的解析器，完成对Spring配置文件的解析），BeanDefinitionHolder（eanDefination的包装类，用来存储BeanDefinition，name以及aliases等）

#### ApplicationContext

>不多介绍了，总之处理特殊Bean和定义基础Bean容器都是由它衍生出来的，内含Bean工厂、应用事件、资源加载、国际化相关接口，并有众多的实现类！

#### 总结

> 分析完之后，鱼皮大大的架构图可以对应相关模块如下

![spring-framework-ioc-source-71](C:\Users\yato\Desktop\U know wt\面试准备\dailyNote\笔记.assets\spring-framework-ioc-source-71.png)

### 初始化流程



### Bean实例化