# FrameWork

> 其实组件不止下述这些，个人编写简易Spring以及简易SpringMvc项目中略过了一些非必要但也很重要的模块。想快速了解可以看看[Spring FrameWork 5](https://pdai.tech/md/spring/spring-x-framework-introduce.html)专栏，鱼皮大大写的。想深入了解的话推荐看陈雄华老师在电子工业出版社出版的Spring有关书籍，讲的很细，日后希望可以详读。

## Core Container（Spring核心容器）

> 实现其他模块的基础，由Beans模块、Core核心模块、Context上下文模块和SpEL表达式语言模块组成。想在项目中实现AOP等功能必须仰赖于该模块。下面介绍一下SpEL模块外的三个模块

- **Beans 模块**：提供了框架的基础部分，包括控制反转和依赖注入。
- **Core 核心模块**：封装了 Spring 框架的底层部分，包括资源访问、类型转换及一些常用工具类。
- **Context 上下文模块**：建立在 Core 和 Beans 模块的基础之上，集成 Beans 模块功能并添加资源绑定、数据验证、国际化、Java EE 支持、容器生命周期、事件传播等。ApplicationContext 接口是上下文模块的焦点。

## Aspects

> 提供与 AspectJ 的集成，是一个功能强大且成熟的面向切面编程（AOP）框架。（和Spring AOP实现上没有关系，只是Spring可以集成AspectJ，使用相关方法）

## IOC

> 控制反转，反转的是依赖对象的获取。由容器创建对象并且进行管理（在合适的时候注入），取代了本来由我们自己创建获取对象的操作。在Spring Boot中使用@Autowired注解就搞定对象注入，在项目中得深入看看背后的过程了。

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

> 总体来说功能如下图所示嘞：

![IOC功能](笔记.assets\spring-framework-ioc-source-7.png)

1. 加载Bean配置：对不同类型资源的加载（比如xml配置文件），解析成生成统一Bean的配置（Bean容器，对Bean进行定义和行为管理）
2. 根据Bean定义加载生成Bean实例，并放置在Bean容器中：比如Bean依赖注入、Bean嵌套、Bean缓存
3. 特殊Bean的处理：比如国际化Message等生成特殊类结构支撑
4. 对容器中Bean提供统一管理和调用：一般来说采用工厂模式

### 体系结构

> 主要分为三大块！：
> 
> 1. BeanFactory和BeanRegistry：定义IOC容器功能规范和Bean的注册
> 2. BeanDefinition：定义各种Bean对象及其相互的关系
> 3. ApplicationContext：IOC容器的接口类，在基本IoC容器实现基础上实现访问资源、国际化、应用事件等功能

#### BeanFactory

> 列出实现的方法如下：

1. 根据bean名字和Class类型来获得bean实例
2. 返回指定bean的Provider
3. 检查工厂中是否包含给定name的bean，或者外部注册的bean
4. 检查所给定name的bean是否为单例/或原型
5. 判断所给name的类型与type是否匹配
6. 获取给定name的bean的类型
7. 返回给定name的bean的别名（Aliases）

#### BeanRegistry

> Spring配置文件中的每一个`bean`在Spring容器中都会通过BeanDefinition对象表示。BeanDefinitionRegistry接口提供向容器手工注册BeanDefinition对象的方法

#### BeanDefinition

> 对Bean对象和对象间关系进行定义，相关的还有BeanDefinitionReader（BeanDefinition的解析器，完成对Spring配置文件的解析），BeanDefinitionHolder（eanDefination的包装类，用来存储BeanDefinition，name以及aliases等）

#### ApplicationContext

> 继承自BeanFactory，实现定义基础Bean容器功能基础上额外拓展处理特殊Bean的能力，内含Bean工厂、应用事件、资源加载、国际化相关接口，有众多的扩展实现。

#### 总结

> 分析完之后，鱼皮大大的架构图可以对应相关模块如下

![spring-framework-ioc-source-71](笔记.assets\spring-framework-ioc-source-71.png)

个人认为，BeanFactory面向的是Spring开发者，而ApplicationContext面向的是Spring使用者。这也是封装的含义所在 

### 初始化流程

Spring IoC容器载入定义资源从refresh()函数开始

> Spring实现将资源配置通过加载、解析、生成BeanDefinition注册到IoC容器（**本质上是存放BeanDefinition的ConcurrentHashMap<String,Object>**）的过程

**从源码上看，具体实现为**

1. 调用父类构造方法为容器设置好Bean资源加载器：
   
   1. 调用默认构造函数初始化容器id，name，状态以及资源解析器
   2. 调用setParent方法将父容器的Environment合并到当前容器

2. 设置配置路径进行Bean定义资源文件定位
   
   1. 调用setConfigLocations方法从Environment中解析Bean配置文件路径

3. 初始化容器，调用模板方法refresh（该模板提供钩子方法）
   
   1. 创建IoC容器前，如果有容器存在，需要将已有容器销毁和关闭
   2. 建立新的Ioc容器

**从流程上看：**

![spring-framework-ioc-source-9](C:\Users\yato\Desktop\U know wt\面试准备\dailyNote\笔记.assets\spring-framework-ioc-source-9.png)

1. 调用refresh方法进入初始化入口
2. 调用loadBeanDefinition方法载入beanDefinition
   1. 通过ResourceLoader实现资源文件位置定位
   2. 通过BeanDefinitionReader完成定义信息的解析和Bean信息的注册
   3. 实现BeanDefinitionRegistry接口将BeanDefinition注册到IoC容器中
3. 通过BeanFactory和ApplicationContext使用Spring的IoC服务

> 这里补充一下关于模板方法模式的一些知识
> 
> **模板方法是一个算法骨架，是一系列调用的基本方法，而基本方法可以分为：**
> 
> 1. 抽象方法：声明但未实现
> 2. 具体方法：实现，可以重写或继承
> 3. 钩子方法：实现，分为用于判断的逻辑方法，和需要子类重写的空方法两种
> 
> **在IoC容器初始化过程中用到的钩子方法有**
> 
> 1. prepareRefresh：对应BeanFactory、MessageSource、ApplicationEvent的初始化
> 2. onRefresh：注册监听器
> 3. finishRefresh：ioc容器初始化完成
> 4. cancelRefresh：销毁已初始化的ioc容器（这个方法是前序方法出错调用的

### Bean实例化

#### getBean(String name)

> 具体流程体现在**getBean**方法，注意的是Spring进行Bean实例化时：会确保依赖也被初始化。根据beanDefinition的信息（单例、原型、bean的scope）有三种不同的创建bean实例方法。

**大概的流程是**：

1. 从beanDefinitionMap通过beanName获得BeanDefinition
2. 从BeanDefinition中获得beanClassName
3. 通过反射初始化beanClassName的实例instance
   1. 构造函数从BeanDefinition的getConstructorArgumentValues()方法获取
   2. 属性值从BeanDefinition的getPropertyValues()方法获取
4. 返回beanName的实例instance

> 上面提到了依赖，这个依赖我们不难想象。也许beanA和beanB存在这样的关系，beanA中有属性beanB，而beanB中也有属性beanA。正常程序中是完全不会有影响的，因为并不会强制要求属性与对象同时进行初始化。但是上文步骤4中提到，Spring源码中会要求依赖也被确保初始化！wow，here is the problem，come to solve it！

#### 三级缓存

> 三级缓存是Spring为解决单例bean的循环问题而使用的一种策略。具体而言体现在调用**getSingleton**方法，会查找bean是否在缓存中，在哪一级，以及进行相对应的处理

**哪三层？**

1. 一级缓存（singletonObjects）：单例对象缓存池的成熟对象，已经实例化并且属性赋值
2. 二级缓存（earlySingletonObjects）：单例对象缓存池的半成品对象，已经实例化但属性尚未赋值
3. 三级缓存（singletonFactories）：单例工厂的缓存

**方法执行过程？**

1. 先从一级缓存singletonObjects中获取
2. 若是获取不到，而且对象正在建立中，再从二级换从earlySingletonObjects中获取
3. 若是仍然获取不到且容许singletonFactories经过get

**如何解决问题？**

关键在于第三层缓存！Spring在调用createBeanInstance后，即使对象没有进行完全的初始化（根据对象引用已经可以定位到堆中的对象了，但是对象内部的一些属性还没有填充（不知道这个词恰不恰当，暂时这么说）），也会经过ObjectFactory将对象存入三级缓存中。如果存在循环依赖，当对象B在三级缓存中读到该对象（**需要Spring允许调用getObject方法**），对象B就可以进行初始化。当对象B创建完毕后，将本身放到一级缓存中，A对象就可以接着初始化

**解决不了哪些问题？**

1. 构造器的循环依赖：调用构造方法之前无法将对象放入三级缓存中，策略失效
2. protype作用域循环依赖：Spring不缓存protype作用域bean，策略失效
3. 多例的循环依赖：多实例bean每调用一次getBean都会执行一次构造方法，没有三级缓存

> 这里提一下上述对象怎么解决：
> 
> 1. 生成代理对象产生的循环依赖可以采用延迟加载，或指定加载先后顺序解决
> 2. 使用@DependsOn产生的循环依赖，需要找到对应地方迫使其不再循环依赖
> 3. 多例产生循环依赖：将bean改为单例
> 4. 构造器循环依赖：采用延迟加载解决

### Bean生命周期

> 从逻辑上分，可以分为实例化、属性赋值、初始化、销毁几个部分
> 
> 从方法调用而言，涉及到BeanPostProcessor（容器级生命周期接口方法）、Bean自身的方法、Aware接口相关方法（Bean级生命周期接口方法），Spring通过独立于Bean的接口实现类可以实现对Bean生命周期做动态增强的功能

**流程大致如下：**

![Bean生命周期](C:\Users\yato\Desktop\U know wt\面试准备\dailyNote\笔记.assets\Bean生命周期.png)

**BeanFactoryProcessor&BeanPostProcessor**

1. BeanFactoryPostProcessor：spring提供的容器扩展机制，允许我们在bean实例化之前修改bean的定义信息即BeanDefinition的信息。其重要的实现类有PropertyPlaceholderConfigurer和CustomEditorConfigurer，PropertyPlaceholderConfigurer的作用是用properties文件的配置值替换xml文件中的占位符，CustomEditorConfigurer的作用是实现类型转换

2. BeanPostProcessor：spring提供的容器扩展机制，不同于BeanFactoryPostProcessor的是，BeanPostProcessor在bean实例化后修改bean或替换bean。（实例化后再工作，很容易联想到AOP吧！）

## AOP

> 提供了面向切面编程实现，提供比如日志记录、权限控制、性能统计等通用功能和业务逻辑分离的技术，并且能动态的把这些功能添加到需要的代码中，这样各司其职，降低业务逻辑和通用功能的耦合。

简单来说，Spring实现AOP通过动态代理。而动态代理呢，又可以分为JDK、Cglib、Aspects几种。

JDK的动态代理的缺点是只能对接口作代理，其原理是反射。

而Cglib则可以对任何类进行代理，其原理是字节码增强，更灵活更高效，但是会更复杂。（不过Spring屏蔽了复杂性，只是原理复杂

# 实现思路

## 1.实现简易的Bean容器

> Spring包含并管理应用对象的配置和生命周期，Bean通过零件化拆分存放至BeanDefinitoin的方式，让Spring更加容易管理Bean对象，并可以统一进行装配(包括Bean初始化、属性填充等)

**如何实现一个Spring容器**(BeanFactory)

简单的方法就是用Map，可以用HashMap也可以用ConcurrentHashMap。项目中采用HashMap，实质上就是存储Bean名称和BeanDefinition的映射

## 2.实现Bean的定义、注册、获取

定义：BeanDefinition，存储Bean类信息

注册：将定义存储在容器中

获取：通过单例模式获取Bean的实例(需要判断bean是否在缓存中，并根据相关情况进行实例化操作)

**模板模式**：

Bean工厂提供Bean获取方法，并由AbstratBeanFactory抽象类进行实现。统一接口的通用核心方法的调用逻辑和标准定义！类的继承者只需要关心具体方法的逻辑实现

## 3.提供对象实例化策略

如果是对于没有属性的bean来说(大部分常见bean都是无参的)，通过beanClass.newInstance就已经完成任务了，但是对于有属性的bean而言，实例化分成两种流派，各有千秋

1. 无参构造：不用获取Constructor，直接创建实例，对于jdk而言是newInstance，对于cglib而言是enhancer.create
2. 有参构造：需要获取对应的Constructor，通过对应的构造器来进行实例化

#### 构造函数

需要获取bean的Constructor，并与实例化所给的参数进行匹配，在Spring中会检查参数的类型和个数是否都匹配，但是为了简单项目中只会检查参数的个数是否匹配

#### NoOp vs MethodInterceptor

> 一开始用MethodInterceptor作为enhancer的callback一直报错，用了NoOp才解决，稍微探究一番

callback可以认为是cglib生成字节码的实现手段，一共可以分为6种

1. MethodInterceptor：类似于环绕增强
2. FixedValue：替换源方法的返回值
3. InvocationHandler：用于自定义实现，类似于MethodInterceptor
4. LazyLoader：单例类的延迟生成代理
5. Dispatcher：原型类的延迟生成代理
6. NoOp：啥也不干

实现MethodInterceptor接口时，需要实现intercept方法，参数为对象、方法、方法参数、方法代理

## 4.属性填充、依赖注入

> 这里的属性更偏向于成员变量的意思，实质上属性的官方定义是指get或set方法去掉get或set后剩下字母。比如 getSize 即有属性size

**实现**：在BeanDefinition中添加属性相关内容，以List+Map形式存储bean类的成员变量名称和类型。进行属性填充即通过BeanDefinitoin中的属性信息与bean类的属性信息进行匹配，将BeanDefinition存储的属性填充进bean实例中

**注意事项**：进行属性填充时需要注意该属性是否为对象类型，单独对对象的依赖进行管理。在注入对象依赖时，需要先对注入的对象进行实例化

## 5.资源加载器解析文件注册对象

> 一般来说Spring有配置文件就可以实例化bean，不用手动创建。所以我们需要一个资源解析器，可以读取classpath、本地文件、云文件中的配置内容(xml)，并且可以对配置文件进行解析，将Bean对象注册到Spring容器中

**实现**：首先是定义资源和资源解析器。具体来说实现有classpath资源、文件系统资源和Url资源，资源解析器的默认实现负责读取这三种资源。随后实现对xml文件的解析，主要是根据读取标签相关值，根据标签名进行相应处理，生成bean元信息(beanDefinition)

### org.w3c.dom

org.w3c.dom为DOM(文档对象模型)提供接口

Document(文档树模型)则可以表示整个HTML或XML文档

Node是DOM中的基本对象，代表文档树的抽象节点(标签啦)

Element是Node类最主要的子对象，可以存取属性

### CharSequence

> 使用hutool的StrUtil建议使用CharSequenceUtil，有啥不一样呢？

CharSequence是一个接口，表示有序的字符集合，提供一些基本的操作方法

String、StringBuilder、StringBuffer都实现该接口

**较之于String**：可读可写；泛用性更强

## 6.应用上下文：超级BeanFactory

> ApplicationContext：把相应的XML加载、注册、实例化以及新增的修改和扩展都直接实现
>
> 方便、快捷，是Spring的对外接口

**refresh方法**:

1. 加载：创建BeanFactory，从Xml文件加载BeanDefinition，获取BeanFactory
2. 修改：Bean实例化前，可以进行BeanDefintion的修改
3. 注册：BeanPostProcessor提前于其他对象实例化之前执行注册操作
4. 实例化：Bean对象(BeanPostProcessor)
	1. 前置处理
	2. CreateBean
	3. 后置处理

**BeanFactoryPostProcessor**：

实现在所有BeanDefinition加载完成后，实例化Bean对象之前，提供修改BeanDefinition属性的机制

**BeanPostProcessor**：

修改实例化Bean对象的扩展点，可以在Bean对象执行初始化方法前后执行相关方法

## 7.初始化和销毁方法

> Spring中bean可以读取在xml文件中自定义的初始化和销毁方法名，调用实现接口的对应方法

**InitalizingBean、DisposableBean**：

定义初始化方法和销毁方法的接口，想实现初始化和销毁方法的bean可以实现这两个接口

当然，并不是所有的初始化和销毁方法都需要自己定义去实现，肯定是需要进行一些初步的具体实现的。DisposableBean的初步实现就由DisposableBeanAdapter适配器完成，这里涉及到了适配器模式，让方法有了更多的延展性

### 8.Aware感知容器对象

> 要获得Spring框架提供的BeanFactory、ApplicationContext、BeanClassLoader等等进行框架扩展使用时，涉及到对容器操作的感知

#### Aware

> 感知标记性接口，具体子类定义和实现可以感知容器中相关对象

实现有：BeanFactoryAware等等



# 心得体会

## 设计模式

### 模板方法

> 模板方法会定义一个算法的骨架，将一些步骤延迟到子类去实现。使得在子类不改变算法结构的基础上，可以重新定义算法中的某些步骤

#### 使用

使用该设计模式的类中，一般有如下几种方法

1. 抽象方法：抽象类声明，由具体子类实现，并以abstract关键字进行标识
2. 具体方法：抽象类声明并实现，子类不Override
3. 钩子方法：抽象类声明并实现，子类可以Override。通常抽象类会给出没有具体实现的空方法
4. 模板方法：定义算法逻辑骨架的方法

在代码实现中，refresh方法就是一个典型的模板方法

```java
public void refresh() throws BeansException {
    // 1.创建BeanFactory，并加载BeanDefinition
    refreshBeanFactory();
    // 2.获取BeanFactory
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    // 3.Bean实例化之前，执行BeanFactoryPostProcessor
    invokeBeanFactoryPostProcessors(beanFactory);
    // 4.BeanPostProcessor 需要提前与其他Bean对象实例化之前进行注册操作
    registerBeanPostProcessors(beanFactory);
    // 5.提前实例化单例对象
    beanFactory.preInstantiateSingletons();
}
```

refreshBeanFactory和getBeanFactory是抽象方法

```java
protected abstract void refreshBeanFactory() throws BeansException;

protected abstract ConfigurableListableBeanFactory getBeanFactory();
```

invokeBeanFactoryPostProcessors和registerBeanPostProcessors是具体方法

#### 优点/应用场景

个人认为使用该设计模式可以在以达成某一特定目标为前提时，可以在一个整体的框架之下，有不同具体的实现方式。不仅

### 简单工厂/工厂模式

### 代理模式

> 代理模式是

### 单例模式

> 单例模式是

### 适配器模式

> 适配器模式指的是

#### 使用

销毁方法多种，是AbstractApplicationContext注册虚拟机之后，虚拟机关闭之前执行的动作。在销毁方法执行时不关注底层的具体实现，通过统一接口进行销毁，于是使用适配器`DisposableBeanAdaptor`，实现统一的处理

## 相关技术

### Hook

> 一种消息拦截机制，并能够对拦截信息进行自动处理
>
> 1. 线程钩子：可以拦截单个进程的消息
> 2. 系统钩子：能够拦截全部进程的消息

使用方法(使用场景在方法备注中)

```java
// 1.程序正常退出，最新的非守护线程退出或该退出方法被唤醒
// 2.JVM被中断
Runtime.getRuntime().addShutdownHook(Thread）；
```

测试一下Hook在Java中的使用！

```java
@Slf4j
public class HookTest {
    private void selfIntroduction() {
        log.info(this.getClass().getSimpleName());
    }
    @Test
    public void testHook() {
        // 程序退出时候调用selfIntroduction
        Runtime.getRuntime().addShutdownHook(new Thread(this::selfIntroduction));
        log.info("do nothing");
    }
}
```

```txt
15:49:15.394 [main] INFO edu.neu.spring.test.HookTest - do nothing
15:49:15.394 [Thread-0] INFO edu.neu.spring.test.HookTest - HookTest
```

### HuTool



# 参考资料

1. 《手撸Spring》-小傅哥
2. https://github.com/code4craft/tiny-spring
3. https://github.com/DerekYRC/mini-spring