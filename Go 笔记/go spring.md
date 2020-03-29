[TOC]



# Go-Spring : Another Go Style！

https://juejin.im/post/5d50f403e51d4561e84fcbfa



## Go-Spring 核心原理 (上)

https://mp.weixin.qq.com/s?__biz=MzU2NzQ2MDkwMQ==&mid=2247484411&idx=1&sn=2c69789e7e3d803ef25dcb006ae75447&chksm=fc9d96b1cbea1fa7fb9c6423ae3e8e86bc0fe8623bc8031c26169c4dece657cc4ca954bc5fd4&mpshare=1&scene=23&srcid=&sharer_sharetime=1570614780451&sharer_shareid=c26dbeef2e683a248815324aa6ec6ee2#rd





## google wire

https://github.com/google/wire





# 源码解读

## 项目：app-starter

### app.starter.go

#### AppRunner（interface）

```go
// 应用执行器
//
type AppRunner interface {
	Start()    // 启动执行器
	ShutDown() // 关闭执行器
}

var exitChan chan struct{}

// 启动应用执行器。
//
func Run(runner AppRunner) {

	exitChan = make(chan struct{})

	go func() {
		// 响应控制台的 Ctrl+C 及 kill 命令。

		sig := make(chan os.Signal, 1)
		signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)

		<-sig

		SafeCloseChan(exitChan)
	}()

	runner.Start()

	<-exitChan

	runner.ShutDown()
}

// 退出当前的应用程序。
//
func Exit() {
	SafeCloseChan(exitChan)
}

func SafeCloseChan(ch chan struct{}) {
	select {
	case <-ch:
		// Already closed. Don't close again.
	default:
		// Safe to close here.
		close(ch)
	}
}
```



## 项目：go-spring-boot

### spring-boot-app.go

#### Application（struct）

Application 实现了 AppRunner 接口

```go
// 定义 SpringBoot 应用
//
type Application struct {  
	AppContext     ApplicationContext // 应用上下文
	ConfigLocation string             // 配置文件目录
	ConfigParsers  []ConfigParser     // 配置文件解析器
}

// 启动 SpringBoot 应用
//
func (app *Application) Start() {

	// 加载配置文件
	if err := app.loadConfigFiles(); err != nil {
		panic(err)
	}

	// 注册 ApplicationContext Bean 对象
	app.AppContext.RegisterBean(app.AppContext)

	// 初始化所有的 SpringBoot 模块
	for _, fn := range Modules {
		fn(app.AppContext)
	}

	// 依赖注入
	if err := app.AppContext.AutoWireBeans(); err != nil {
		panic(err)
	}

	// 通知应用启动事件
	var eventBeans []ApplicationEvent
	app.AppContext.FindBeansByType(&eventBeans)

	if eventBeans != nil && len(eventBeans) > 0 {
		for _, bean := range eventBeans {
			bean.OnStartApplication(app.AppContext)
		}
	}
}

// 加载应用配置文件
//
func (app *Application) loadConfigFiles() error {
  
}

// 停止 SpringBoot 应用
//
func (app *Application) ShutDown() {

	// 通知应用停止事件
	var eventBeans []ApplicationEvent
	app.AppContext.FindBeansByType(&eventBeans)

	if eventBeans != nil && len(eventBeans) > 0 {
		for _, bean := range eventBeans {
			bean.OnStopApplication(app.AppContext)
		}
	}

	// 等待所有 goroutine 退出
	app.AppContext.Wait()

	fmt.Println("spring boot exit")
}
```

### spring-boot-context

#### ApplicationContext（interface）

```go
// Application 上下文
//
type ApplicationContext interface {
	// 继承 SpringContext 的功能
	SpringCore.SpringContext

	// 安全的启动一个 goroutine
	SafeGoroutine(fn GoFunc)

	// 等待所有 goroutine 退出
	Wait()
}

// ApplicationContext 的默认版本
//
type DefaultApplicationContext struct {
	*SpringCore.DefaultSpringContext

	wg sync.WaitGroup
}

// 工厂函数
//
func NewDefaultApplicationContext() *DefaultApplicationContext {
	return &DefaultApplicationContext{
		DefaultSpringContext: SpringCore.NewDefaultSpringContext(),
	}
}

// ApplicationContext 接口的实现
// ......
// ......
```

## 项目：go-spring-boot

### springcore\spring-context.go

#### SpringContext（interface）

```go
// 定义 SpringContext 接口
//
type SpringContext interface {
	// 注册 SpringBean 使用默认的 Bean 名称
	RegisterBean(bean SpringBean)

	// 注册 SpringBean 使用指定的 Bean 名称
	RegisterNameBean(name string, bean SpringBean)

	// 根据 Bean 类型查找 SpringBean 数组
	FindBeansByType(i interface{})

	// 根据 Bean 类型查找 SpringBeanDefinition 数组
	FindBeanDefinitionsByType(t reflect.Type) []*SpringBeanDefinition

	// 根据 Bean 名称查找 SpringBean
	FindBeanByName(name string) SpringBean

	// 根据 Bean 名称查找 SpringBeanDefinition
	FindBeanDefinitionByName(name string) *SpringBeanDefinition

	// 获取属性值
	GetProperties(name string) interface{}

	// 设置属性值
	SetProperties(name string, value interface{})

	// 获取指定前缀的属性值集合
	GetPrefixProperties(prefix string) map[string]interface{}

	// 获取属性值，如果没有找到则使用指定的默认值
	GetDefaultProperties(name string, defaultValue interface{}) (interface{}, bool)

	// 自动绑定所有的 SpringBean
	AutoWireBeans() error

	// 绑定外部指定的 SpringBean
	WireBean(bean SpringBean) error
}
```

#### DefaultSpringContext （struct）

```go
// SpringContext 的默认版本
//
type DefaultSpringContext struct {
	beanDefinitionMap map[string]*SpringBeanDefinition // Bean 集合
	propertiesMap     map[string]interface{}           // 属性值集合
}

// 工厂函数
//
func NewDefaultSpringContext() *DefaultSpringContext {
	return &DefaultSpringContext{
		beanDefinitionMap: make(map[string]*SpringBeanDefinition),
		propertiesMap:     make(map[string]interface{}),
	}
}

// SpringContext 接口的实现
// ......
// ......
```

#### SpringBean（interface）

```go
// 定义 SpringBean 类型
//
type SpringBean interface{}

// SpringBean 初始化接口
//
type SpringBeanInitialization interface {
	InitBean(ctx SpringContext)
}

// 定义 SpringBeanDefinition 类型
//
type SpringBeanDefinition struct {
	Bean  SpringBean
	Name  string
	Init  int
	Type  reflect.Type
	Value reflect.Value
}
```

