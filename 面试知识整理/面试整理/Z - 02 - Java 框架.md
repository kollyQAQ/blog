[TOC]

## Spring

### Spring 中的 BeanFactory 和 ApplicationContext 是什么关系？

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/ApplicationContext 集成关系图.png)

ApplicationContext 接口是由 BeanFactory 接口派生出来的，所以提供了 BeanFactory 的所有功能，提供了更多的扩展功能

**两者对比：**

- `BeanFactory` ：延迟注入 (使用到某个 bean 的时候才会注入), 相比于 `BeanFactory` 来说会占用更少的内存，程序启动速度更快。
- `ApplicationContext` ：容器启动的时候，不管你用没用到，一次性创建所有 bean 。`BeanFactory` 仅提供了最基本的依赖注入支持，`ApplicationContext` 扩展了 `BeanFactory` , 除了有 `BeanFactory` 的功能还有额外更多功能，所以一般开发人员使用 `ApplicationContext` 会更多。

### Spring IOC 原理

#### ApplicationContext（BeanFactory） 的生命流程

1. ApplicationContext 加载 Bean 配置文件，将读到的 Bean 配置封装成 BeanDefinition 对象
2. 将封装好的 BeanDefinition 对象注册到 BeanDefinition 容器中
3. 注册 BeanPostProcessor 相关实现类到 BeanPostProcessor 容器中
4. ApplicationContext 进入就绪状态
5. 外部调用 ApplicationContext 的 getBean (String name) 方法，ApplicationContext 着手实例化相应的 bean（详见 Spring Bean 的生命流程）
6. 重复步骤 3 和 4，直至程序退出，ApplicationContext 被销毁

上面简单罗列了 ApplicationContext 的生命流程，也就是 IOC 容器的生命流程

#### Spring Bean 的生命流程

[![bean实例化过程](https://blog-pictures.oss-cn-shanghai.aliyuncs.com/bean%e5%ae%9e%e4%be%8b%e5%8c%96%e8%bf%87%e7%a8%8b.png)](http://www.coolblog.xyz/)

1. 通过反射实例化 bean 对象，类似于 new XXObject()
2. 将配置文件中配置的属性填充到刚刚创建的 bean 对象中。
3. 检查 bean 对象是否实现了 Aware 一类的接口，如果实现了则把相应的依赖设置到 bean 对象中。比如如果 bean 实现了 BeanFactoryAware 接口，Spring 容器在实例化bean的过程中，会将 BeanFactory 容器注入到 bean 中。
4. 调用 BeanPostProcessor 前置处理方法，即 postProcessBeforeInitialization(Object bean, String beanName)。
5. 检查 bean 对象是否实现了 InitializingBean 接口，如果实现，则调用 afterPropertiesSet 方法。或者检查配置文件中是否配置了 init-method 属性，如果配置了，则去调用 init-method 属性配置的方法。
6. 调用 BeanPostProcessor 后置处理方法，即 postProcessAfterInitialization(Object bean, String beanName)。**我们所熟知的 AOP 就是在这里将 Adivce 逻辑织入到 bean 中的**。
7. 注册 Destruction 相关回调方法。
8. bean 对象处于就绪状态，可以使用了。
9. 应用上下文被销毁，调用注册的 Destruction 相关方法。如果 bean 实现了 DispostbleBean 接口，Spring 容器会调用 destroy 方法。如果在配置文件中配置了 destroy 属性，Spring 容器则会调用 destroy 属性对应的方法。

####  Spring AOP 织入流程

AOP 是基于动态代理模式实现的，具体实现上可以基于 JDK 动态代理或者 Cglib 动态代理。其中 JDK 动态代理只能代理实现了接口的对象，而 Cglib 动态代理则无此限制。所以在为没有实现接口的对象生成代理时，只能使用 Cglib

```java
public interface BeanPostProcessor {

    Object postProcessBeforeInitialization(Object bean, String beanName) throws Exception;

    Object postProcessAfterInitialization(Object bean, String beanName) throws Exception;
}
```

1. 查找实现了 PointcutAdvisor 类型的切面类，切面类包含了 Pointcut 和 Advice 实现类对象。
2. 检查 Pointcut 中的表达式是否能匹配当前 bean 对象。
3. 如果匹配到了，表明应该对此对象织入 Advice。
4. 将 bean，bean class 对象，bean 实现的 interface 的数组，Advice 对象传给代理工厂 ProxyFactory。代理工厂创建出 AopProxy 实现类，最后由 AopProxy 实现类创建 bean 的代理类，并将这个代理类返回。此时从 postProcessAfterInitialization (Object bean, String beanName) 返回的 bean 此时就不是原来的 bean 了，而是 bean 的代理类。原 bean 就这样被无感的替换掉了，是不是有点偷天换柱的感觉。

Spring AOP 生成代理类的逻辑是在 AbstractAutoProxyCreator 相关子类中实现的

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/AOP 原理 yuanli.png)

### Spring AOP 的原理?

**Spring AOP 就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 **JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 **Cglib** ，这时候 Spring AOP 会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

![SpringAOPProcess](https://images.xiaozhuanlan.com/photo/2019/deddb073e946b24059dd658f15dfda09.jpg)

当然你也可以使用 AspectJ ，Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。

使用 AOP 之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了 AOP 。

#### Spring AOP 和 AspectJ AOP 有什么区别？

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理 (Proxying)，而 AspectJ 基于字节码操作 (Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比 Spring AOP 快很多。

### Spring @Transactional 事务实现机制

- 在应用系统调用声明了 @Transactional 的目标方法时，Spring Framework 默认使用 AOP 代理（Spring AOP 代理有 CglibAopProxy 和 JdkDynamicAopProxy 两种），在代码运行时生成一个代理对象，根据 @Transactional 的属性配置信息，这个代理对象决定该声明 @Transactional 的目标方法是否由拦截器 TransactionInterceptor 来拦截，会在目标方法开始执行之前创建并加入事务，并执行目标方法的逻辑, 最后根据执行情况是否出现异常，利用抽象事务管理器 AbstractPlatformTransactionManager 操作数据源 DataSource 提交或回滚事务
- Spring AOP 代理有 CglibAopProxy 和 JdkDynamicAopProxy 两种。以 CglibAopProxy 为例，对于 CglibAopProxy，需要调用其内部类的 DynamicAdvisedInterceptor 的 intercept 方法。对于 JdkDynamicAopProxy，需要调用其 invoke 方法。
- ![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/Spring.png)
- https://blog.csdn.net/jiangyu1013/article/details/84400886`

### 谈谈Spring中都用到了那些设计模式?

工厂，单例，模版，观察者，装饰者

## Mybatis

### 大致步骤

1. 读取配置文件
2. 创建 SqlSessionFactoryBuilder 对象
3. 通过 SqlSessionFactoryBuilder 对象创建 SqlSessionFactory
4. 通过 SqlSessionFactory 创建 SqlSession
5. 为 Dao 接口生成代理类
6. 调用接口方法访问数据库

### Mybatis 的缓存是怎么做的?
https://tech.meituan.com/2018/01/19/mybatis-cache.html
http://www.tianxiaobo.com/2018/08/25/MyBatis-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E7%BC%93%E5%AD%98%E5%8E%9F%E7%90%86/#1%E7%AE%80%E4%BB%8B

## Spring Boot

### 自动配置如何实现？



## Spring Cloud

### 有哪些组件？



## Dubbo

### Dubbo的原理

- 第一步：provider 向注册中心去注册
- 第二步：consumer 从注册中心订阅服务，注册中心会通知 consumer 注册好的服务
- 第三步：consumer 调用 provider
- 第四步：consumer 和 provider 都异步通知监控中心

![](https://kolly-imgstore.oss-cn-shenzhen.aliyuncs.com/img/Dubbo Architecture.png)


Provider: 暴露服务的**服务提供方**。

Consumer: 调用远程服务的**服务消费方**。

Registry: 服务注册与发现的**注册中心**。

Monitor: 统计服务的调用次调和调用时间的**监控中心**。

Container: 服务**运行容器**。

### 注册中心挂了可以继续通信吗？

可以，因为刚开始初始化的时候，消费者会将提供者的地址等信息**拉取到本地缓存**，所以注册中心挂了可以继续通信。

### SPI 扩展机制如何实现的
SPI 全称为 Service Provider Interface，是一种服务发现机制
http://www.tianxiaobo.com/2018/10/01/Dubbo-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-SPI-%E6%9C%BA%E5%88%B6/

SPI 注解 @SPI

ExtensionLoader 的 getExtensionLoader 方法获取一个 ExtensionLoader 实例，然后再通过 ExtensionLoader 的 getExtension 方法获取拓展类对象

```java
Protocol protocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
```

Protocol 接口，在系统运行的时候，，dubbo 会判断一下应该选用这个 Protocol 接口的哪个实现类来实例化对象来使用。

它会去找一个你配置的 Protocol，将你配置的 Protocol 实现类，加载到 jvm 中来，然后实例化对象，就用你的那个 Protocol 实现类就可以了。

上面那行代码就是 dubbo 里大量使用的，就是对很多组件，都是保留一个接口和多个实现，然后在系统运行的时候动态根据配置去找到对应的实现类。如果你没配置，那就走默认的实现好了，没问题。

### dubbo和 spring cloud 的区别和优劣势?

Dubbo 是一个分布式服务框架，致力于提供高性能和透明化的 **RPC 远程服务调用方案**，以及 **SOA 服务治理方案**。简单的说，Dubbo 就是个  RPC 框架，说白了就是个**远程服务调用的分布式框架**

### 为什么 Dubbo 比 Spring Cloud 性能要高一些？

因为 Dubbo 采用单一长连接和 NIO 异步通讯（保持连接/轮询处理），使用自定义报文的 TCP 协议，并且序列化使用定制 Hessian2 框架，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况，但不适用于传输大数据的服务调用。而 Spring Cloud 直接使用 HTTP 协议（但也不是强绑定，也可以使用 RPC 库，或者采用 HTTP 2.0 + 长链接方式（Fegin 可以灵活设置））。

https://www.cnblogs.com/xishuai/p/dubbo-and-spring-cloud.html