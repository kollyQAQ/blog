# 什么是值类型和引用类型？ 

值类型保存的是数据本身，引用类型保存的则是数据的指针(或者叫引用、地址都行)。 像 int、bool 等基础类型都是值类型，字符串也是值类型，结构体也是值类型。 假如对 int 等类型重定义并加入成员方法之后，新类型也是值类型吗？是的。换句话说，判断一个类型是值类型还是引用类型依据是它的 kind。 除了指针是引用类型之外，数组、集合、通道等也都是引用类型，这些类型都不直接保存数据，而是保存数据的地址。 由此可见，**多个引用类型的对象是可以共享同一个数据的，而 Bean 指的就是这些可以被共享的数据**。

# Bean 的三种定义形式 

根据定义形式的不同可以将 **Bean 分为三种，分别是对象 Bean、构造函数 Bean 和成员方法 Bean，后两者又称为函数 Bean**。 对象 Bean 保存的是 Bean 的原始数据，函数 Bean 保存的则是创建 Bean 的函数，Go-Spring 会在合适的时机调用这些函数创建 Bean 的数据。 顾名思义，构造函数 Bean 保存的是可以创建 Bean 的构造函数(或者叫工厂函数)，而成员方法 Bean 保存的则是可以创建 Bean 的成员方法。这两者的不同之处在于成员方法 Bean 必须和一个与成员方法的接收者类型相同的 Bean 进行绑定。 此外，函数 Bean 的函数不仅支持可变参数，还支持 Option 模式，这在复用现有第三方代码时会非常方便。 函数 Bean 的函数是不是必须返回引用类型呢？不是的。Go-Spring 在检测到函数的返回值是值类型的时候会自动将其转换成引用类型。 函数 Bean 的函数可以返回接口类型吗？是的。而且 Go-Spring 推荐尽可能返回接口类型，这不仅在代码层面有好处，在 Bean 注册和注入的时候也更方便。 另外，为了简称大部分情况下函数 Bean 指的是构造函数 Bean，而成员方法 Bean 则简称为方法 Bean。这一点需要读者在后续章节注意分辨。

# 什么是 IoC 容器和 Bean 注入

举个例子，假设有一个结构体 Controller，包含了很多叫 Service 的字段， type Controller struct{ userService UserService roleService RoleService deptService DeptService ... } 现在想给 Controller 的这些 Service 字段赋值，传统方式是通过构造函数参数传入或者直接对字段进行赋值。但无论哪一种方式，当 Service 字段变多或者变少之后都会导致修改很多的代码，那么有没有办法可以减少这种修改呢？ 试着将这些 Service 的实例想象成服务提供方，将 Controller 的这些 Service 字段想象成服务消费方，这样只需要服务提供方说明自己的能力，服务消费方说明自己的需求，就可以设计一个服务平台来匹配这种供需关系。 如果这种方案是可行的，稍微想象一下就能感受到这种方式带来的新力量！幸运的是，Go 语言提供了这种可能，通过反射机制很容易就能实现这种新的方式。 在这种方式下，**服务平台就是 IoC 容器，服务消费的过程就是 Bean 注入**。而 Go-Spring 就是 Go 语言中 IoC 容器的一个非常优秀的实现，后续章节会一点一点地揭开它的面纱。

# 初识 SpringContext 

SpringContext 是一个功能完善的 IoC 容器，是 Go-Spring 绝对的核心，其主要功能如下： RegisterBean 和 RegisterNameBean 注册对象 Bean； RegisterBeanFn 和 RegisterNameBeanFn 注册构造函数 Bean；RegisterMethodBean 和 RegisterNameMethodBean 注册成员方法 Bean。 WireBean 对一个外部 Bean 执行注入过程； AutoWireBeans 对已注册的 Bean 执行注入过程。 GetBean 和 GetBeanByName 获取一个 Bean，并确保已完成注入过程； FindBean 和 FindBeanByName 查找一个 Bean，但不保证已完成注入过程； CollectBeans 收集所有符合条件的 Bean，甚至包括符合条件的数组项，并确保均已完成注入过程。 Close 关闭 IoC 容器，并向已注册的 Bean 发送 destroy 消息，用于执行资源清理等工作。 实现 context.Context 接口，用于控制 goroutine 的生命周期。 实现 Properties 接口，用于实现一系列和属性相关的操作，后面会有专门的章节详细讲解。

# 实战对象Bean（1）

![img](https://mmbiz.qpic.cn/mmbiz_png/gxd4s9bVZ1UMXdAtiaeOdlelZX4d2qWuG34xsoUY2sW8eVWPM5Y0ZBF4gtkOZtBVic0dKdW8qCDGNQPqeDmLXcAw/0?wx_fmt=png)

# 实战对象 Bean (2)



![img](https://mmbiz.qpic.cn/mmbiz_png/gxd4s9bVZ1WEAEPA80SC1u6EzChYejzdEmPR7BkxEA2Gu4WqVSgicpPEZOcQBicuFKh49kNBk1YZp7gLYx6D3wvQ/0?wx_fmt=png)

# 实战对象 Bean (3)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/gxd4s9bVZ1U8epteNvicQkm1FpAv27rEg5Bwf99aLbgqd9BFC0hJjuxIdU65RicjIhmbYN4KDxAgK4GD5PA6k8rg/0?wx_fmt=jpeg)