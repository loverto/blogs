# Spring 5 源码分析之Spring技术内幕的读书笔记

读了Spring技术内幕，书中的Spring版本讲的是3.x版本的，这里做个做一个读书笔记，看到少写多少吧，以免到时候忘的干净，虽然之前已经看过，第一章讲了Spring家族产品包含那些，例如Spring core，spring bean,spring tx,spring mvc
spring android等等，讲了Spring的由来，以及Spring的设计理念。不过接下来第二章比较有意思，开始讲SpringIoc的设计原理，在Spring的Ioc中主要有BeanFactory这个接口，根据这个最原始的容器定义接口分别扩展出不同的接口，

## Spring Ioc容器的实现

这里上一下ioc主要的核心接口设计图

![Spring Ioc 核心接口类图](https://ws1.sinaimg.cn/large/61411417ly1g09vg7b00tj21210cy74t.jpg)

书中主要讲的一个代表性容器是XmlBeanFactory，比过这个容器在5.X时已经设置为不推荐使用的状态，这个类的继承实现类图如下所示，从注释上来看，这个类在3.1时已经不推荐用了，推荐直接用DefaultListableBeanFactory

![XmlBeanFactory类图](https://ws1.sinaimg.cn/large/61411417ly1g09vnrgsjij21140jfab6.jpg)

```java
@Deprecated
@SuppressWarnings({"serial", "all"})
public class XmlBeanFactory extends DefaultListableBeanFactory {

	private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);


	/**
	 * Create a new XmlBeanFactory with the given resource,
	 * which must be parsable using DOM.
	 * @param resource the XML resource to load bean definitions from
	 * @throws BeansException in case of loading or parsing errors
	 */
	public XmlBeanFactory(Resource resource) throws BeansException {
		this(resource, null);
	}

	/**
	 * Create a new XmlBeanFactory with the given input stream,
	 * which must be parsable using DOM.
	 * @param resource the XML resource to load bean definitions from
	 * @param parentBeanFactory parent bean factory
	 * @throws BeansException in case of loading or parsing errors
	 */
	public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
		super(parentBeanFactory);
		this.reader.loadBeanDefinitions(resource);
	}

}

```

可以看到核心代码并不是很多，因为大部分的实现代码都在父类中，这样我们就知道，如果直接用父类该如何操作了。


在讲ApplicationContext容器时，从FileSystemXmlApplicationContext这个具体的实现来说明，ApplicationContext可以完成的功能，

![FileSystemXmlApplicationContext类图](https://ws1.sinaimg.cn/large/61411417ly1g09w09kqzvj21950jot9z.jpg)

本类的核心作用，实例化应用上下文的支持，并且启动IOC的refresh过程

```java
	public FileSystemXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}
```

## 容器的初始化过程

