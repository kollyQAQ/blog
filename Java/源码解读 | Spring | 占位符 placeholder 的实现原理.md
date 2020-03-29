[TOC]

#### 先回顾一下我们在 spring 中是如何使用占位符的

**application.xml**

```xml
<!-- 通过配置PropertyPlaceholderConfigurer的bean来扫描 properties 文件 -->
<bean id="placeHolder" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">  
       <property name="location" >  
             <value>classpath*:*.properties</value >  
       </property>  
</bean>

<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <!-- 基本属性 url、username、password -->
        <property name="url" value="jdbc:mysql://${mysql.host}:${mysql.port}/${mysql.database}"/>
        <property name="username" value="${mysql.username}"/>
        <property name="password" value="${mysql.passwd}"/>
</bean>
```

**app.properties**

```properties
mysql.host=127.0.0.1
mysql.port=3306
mysql.database=db_user
mysql.username=root
mysql.passwd=123456
```

#### PropertyPlaceholderConfigurer类解析
![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/7017140-7c0db8cb7c6b61fc?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

根据类图大概可以看出来spring实现占位符替换主要涉及到`BeanFactoryPostProcessor`接口和`PropertyPlaceholderConfigurer`、`PlaceholderConfigurerSupport`、`PropertyResourceConfigurer`三个类

#### 先来分析BeanFactoryPostProcessor接口

Spring提供了的一种叫做`BeanFactoryPostProcessor`的容器扩展机制。它允许我们在容器实例化对象之前，对容器中的BeanDefinition中的信息做一定的修改(比如对某些字段的值进行修改，这就是占位符替换的根本)。

```java
public interface BeanFactoryPostProcessor {
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

什么意思呢？就是只要向 Spring 容器注入了 `BeanFactoryPostProcessor` 接口的实现类，那么在容器实例化对象之前会先调用这个类的`postProcessBeanFactory` 方法，而`PropertyPlaceholderConfigurer`类正是间接实现了`BeanFactoryPostProcessor`接口才完成了占位符的实现（看上面的类图）

我们看到这个接口的参数是`ConfigurableListableBeanFactory`类的一个对象，实际开发中，我们常使用它的getBeanDefinition()方法获取某个bean的元数据定义：BeanDefinition。它有这些方法： 

![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/7017140-7dfadeaa9729b3fe?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**真正实现来BeanFactoryPostProcessor接口的是PropertyResourceConfigurer类，来看看它如何实现postProcessBeanFactory方法的**

```java
	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		try {
			// 把加载的所有的 properties 文件中的键值对都取出来
			Properties mergedProps = mergeProperties();

			// Convert the merged properties, if necessary.
			convertProperties(mergedProps);

			// Let the subclass process the properties.
			processProperties(beanFactory, mergedProps);
		}
		catch (IOException ex) {
			throw new BeanInitializationException("Could not load properties", ex);
		}
	}
```

容器会首先走到这里的postProcessBeanFactory方法里面，`mergeProperties();` 把加载的所有的 properties 文件中的键值对都取出来保存在一起，接下来主要看`processProperties`方法，这个方法是抽象的，具体在**PropertyPlaceholderConfigurer**类中实现，来看代码

```java
	@Override
	protected void processProperties(ConfigurableListableBeanFactory beanFactoryToProcess, Properties props)
			throws BeansException {

		StringValueResolver valueResolver = new PlaceholderResolvingStringValueResolver(props);
		doProcessProperties(beanFactoryToProcess, valueResolver);
	}
```

我们看到它首先是实例化了一个**PlaceholderResolvingStringValueResolver**对象，先看看**PlaceholderResolvingStringValueResolver**这个类，这个是**PropertyPlaceholderConfigurer**的内部类，代码如下

```java
private class PlaceholderResolvingStringValueResolver implements StringValueResolver {

		private final PropertyPlaceholderHelper helper;

		private final PlaceholderResolver resolver;

		public PlaceholderResolvingStringValueResolver(Properties props) {
			this.helper = new PropertyPlaceholderHelper(
					placeholderPrefix, placeholderSuffix, valueSeparator, ignoreUnresolvablePlaceholders);
			this.resolver = new PropertyPlaceholderConfigurerResolver(props);
		}

		@Override
		public String resolveStringValue(String strVal) throws BeansException {
			String resolved = this.helper.replacePlaceholders(strVal, this.resolver);
			if (trimValues) {
				resolved = resolved.trim();
			}
			return (resolved.equals(nullValue) ? null : resolved);
		}
	}
```

可以看到这个类实现了**StringValueResolver**这个接口，这个类的作用是我们分析到后面再回头来看~

这个对象作为参数传给了**doProcessProperties**方法，来看看`doProcessProperties`这个方法，它是在父类**PlaceholderConfigurerSupport**中实现的，代码如下

```java
protected void doProcessProperties(ConfigurableListableBeanFactory beanFactoryToProcess,
			StringValueResolver valueResolver) {

		BeanDefinitionVisitor visitor = new BeanDefinitionVisitor(valueResolver);

		String[] beanNames = beanFactoryToProcess.getBeanDefinitionNames();
		for (String curName : beanNames) {
			// Check that we're not parsing our own bean definition,
			// to avoid failing on unresolvable placeholders in properties file locations.
			if (!(curName.equals(this.beanName) && beanFactoryToProcess.equals(this.beanFactory))) {
				BeanDefinition bd = beanFactoryToProcess.getBeanDefinition(curName);
				try {
					visitor.visitBeanDefinition(bd);
				}
				catch (Exception ex) {
					throw new BeanDefinitionStoreException(bd.getResourceDescription(), curName, ex.getMessage(), ex);
				}
			}
		}

		// New in Spring 2.5: resolve placeholders in alias target names and aliases as well.
		beanFactoryToProcess.resolveAliases(valueResolver);

		// New in Spring 3.0: resolve placeholders in embedded values such as annotation attributes.
		beanFactoryToProcess.addEmbeddedValueResolver(valueResolver);
	}
```

首先根据传进来的**valueResolver**对象生成了一个**BeanDefinitionVisitor**对象，然后循环遍历 beanFactory 里面的每一个 bean，获取他们的**BeanDefinition**，然后调用了`visitor.visitBeanDefinition(bd);`，所以重点就是**BeanDefinitionVisitor**这个对象的**visitBeanDefinition**方法到底对我们的**BeanDefinition**做了什么羞羞的事情，继续往下看~



截取了**BeanDefinitionVisitor**这个类的比较重要的部分代码

```java
public class BeanDefinitionVisitor {

	private StringValueResolver valueResolver;
	
	public BeanDefinitionVisitor(StringValueResolver valueResolver) {
		Assert.notNull(valueResolver, "StringValueResolver must not be null");
		this.valueResolver = valueResolver;
	}
	
	public void visitBeanDefinition(BeanDefinition beanDefinition) {
		visitParentName(beanDefinition);
		visitBeanClassName(beanDefinition);
		visitFactoryBeanName(beanDefinition);
		visitFactoryMethodName(beanDefinition);
		visitScope(beanDefinition);
		//重点是这行，调用visitPropertyValues
		visitPropertyValues(beanDefinition.getPropertyValues());
		ConstructorArgumentValues cas = beanDefinition.getConstructorArgumentValues();
		visitIndexedArgumentValues(cas.getIndexedArgumentValues());
		visitGenericArgumentValues(cas.getGenericArgumentValues());
	}
	
	protected void visitPropertyValues(MutablePropertyValues pvs) {
		PropertyValue[] pvArray = pvs.getPropertyValues();
		for (PropertyValue pv : pvArray) {
			//调用resolveValue
			Object newVal = resolveValue(pv.getValue());
			if (!ObjectUtils.nullSafeEquals(newVal, pv.getValue())) {
				pvs.add(pv.getName(), newVal);
			}
		}
	}
	
	protected Object resolveValue(Object value) {
		// 省略部分代码
		if (value instanceof String) {
			return resolveStringValue((String) value);
		}
		return value;
	}
	
	protected String resolveStringValue(String strVal) {
		// 省略部分代码
		String resolvedValue = this.valueResolver.resolveStringValue(strVal);
		// Return original String if not modified.
		return (strVal.equals(resolvedValue) ? strVal : resolvedValue);
	}
}
```

首先看到构造方法，他接收一个**StringValueResolver**接口的实现类实例作为成员变量，最终在**resolveStringValue**这个方法中调用`this.valueResolver.resolveStringValue(strVal)`发挥作用，这个后面说

来看看这个很关键的**visitBeanDefinition**方法，`visitPropertyValues(beanDefinition.getPropertyValues());`这一行中`beanDefinition.getPropertyValues()`取出了这个 bean 的所有属性，在这里我们看到了我们熟悉的占位符，神秘的面纱马上揭开，嘿嘿

![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/7017140-0a5bc728e0f27ce7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在**visitPropertyValues**调用了**resolveValue**去取这个属性的值对应的最终的结果，因为占位符是 string 的形式，所以最后会落到**resolveStringValue**方法中，在这个方法中我们可以看到`String resolvedValue = this.valueResolver.resolveStringValue(strVal);`，这里的 **strVal** 就是图中看到的`${mysql.druid.initialSize}`，而`this.valueResolver`正是构造函数传进来的参数，也就是**PlaceholderConfigurerSupport**调用`BeanDefinitionVisitor visitor = new BeanDefinitionVisitor(valueResolver);`作为参数传过来的的**valueResolver**，这个对象是最开始提到的**PlaceholderResolvingStringValueResolver**这个类，所以应该就是这个类的**resolveStringValue**方法把`${mysql.druid.initialSize}`替换成 properties 文件中对应的值的



#### 现在重点就是PlaceholderResolvingStringValueResolver这个类

前面说了这个类是**PropertyPlaceholderConfigurer**的内部类，代码如下

```java
private class PlaceholderResolvingStringValueResolver implements StringValueResolver {

		private final PropertyPlaceholderHelper helper;

		private final PlaceholderResolver resolver;

		public PlaceholderResolvingStringValueResolver(Properties props) {
			this.helper = new PropertyPlaceholderHelper(
					placeholderPrefix, placeholderSuffix, valueSeparator, ignoreUnresolvablePlaceholders);
			this.resolver = new PropertyPlaceholderConfigurerResolver(props);
		}

		@Override
		public String resolveStringValue(String strVal) throws BeansException {
			String resolved = this.helper.replacePlaceholders(strVal, this.resolver);
			if (trimValues) {
				resolved = resolved.trim();
			}
			return (resolved.equals(nullValue) ? null : resolved);
		}
	}
```

通过代码可以看到，占位符的替换主要就是在**PropertyPlaceholderHelper**类的**replacePlaceholders**方法中实现的，这个方法代码有点长就不粘贴出来了，感兴趣的可以自己看看**PropertyPlaceholderHelper**源码，也比较简单，总结起来就是找到参数**strVal**中被`#{`和`}`包含的部分，然后调用传入的**resolver**的**resolvePlaceholder**方法找到对应的值来进行替换（如果有嵌套的会递归替换），来看看这个**PlaceholderResolver**接口的实现类**PropertyPlaceholderConfigurerResolver**。

**PlaceholderResolver**接口是在**PropertyPlaceholderHelper**类中定义的一个内部类接口，**PropertyPlaceholderConfigurer**中定义了内部类**PropertyPlaceholderConfigurerResolver**实现了这个接口，代码如下

```java
private class PropertyPlaceholderConfigurerResolver implements PlaceholderResolver {

		private final Properties props;

		private PropertyPlaceholderConfigurerResolver(Properties props) {
			this.props = props;
		}

		@Override
		public String resolvePlaceholder(String placeholderName) {
			return PropertyPlaceholderConfigurer.this.resolvePlaceholder(placeholderName, props, systemPropertiesMode);
		}
	}
```

resolvePlaceholder实现的逻辑如下，说白了就是去 Properties 对象中去对应的值，这里的 props 就是最开始根据 properties 文件生成的，所以到这里应该就真相大白了，其实还是很简单的。

```java
protected String resolvePlaceholder(String placeholder, Properties props, int systemPropertiesMode) {
		String propVal = null;
		if (systemPropertiesMode == SYSTEM_PROPERTIES_MODE_OVERRIDE) {
			propVal = resolveSystemProperty(placeholder);
		}
		if (propVal == null) {
			propVal = resolvePlaceholder(placeholder, props);
		}
		if (propVal == null && systemPropertiesMode == SYSTEM_PROPERTIES_MODE_FALLBACK) {
			propVal = resolveSystemProperty(placeholder);
		}
		return propVal;
	}
	
protected String resolvePlaceholder(String placeholder, Properties props) {
		return props.getProperty(placeholder);
	}
```