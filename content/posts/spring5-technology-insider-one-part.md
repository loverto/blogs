---
date: "2018-07-25T20:18:57+08:00"
title: Spring 5 源码分析之Spring技术内幕的读书笔记
---
# Spring 5 源码分析之Spring技术内幕的读书笔记

再读Spring技术内幕，由于之前没做笔记，再读一次觉得做个记录很有必要，这里会把我看这本书的所看所想做简单地记录，首先发现书中讲的Spring版本太老了，书中的Spring版本讲的是3.x版本的，现在Spring版本已经是5.x了，我会把更新部分的内容同样写出来，这里做个做一个读书笔记，看到少写多少吧，以免到时候忘的干净，第一章讲了Spring家族产品包含那些，例如Spring core，spring bean,spring tx,spring mvc
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

Ioc容器的初始化有个入口，入口就是这个refresh()方法，这个入口方法中包含了，Resource定位，载入和注册。
IOC的核心存储数据结构是HashMap
refresh在AbstractApplicationContext 的类中，方法里面主要的内容如下
![refresh](https://ws1.sinaimg.cn/large/61411417ly1g0d9s6r5wjj20m50pxad0.jpg)

这篇博客内容已经不短了，下面的内容，我们在下篇继续说