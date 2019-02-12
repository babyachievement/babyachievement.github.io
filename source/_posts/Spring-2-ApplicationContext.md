---
title: Spring-2-ApplicationContext
date: 2018-12-26 10:28:56
tags: ["Spring"]
---

# ApplicationContext

作为IOC的的核心组件之一，相比于BeanFactory，ApplicationContext还提供了ResourceLoader功能，ApplicationEventPublisher功能以及MessageSource功能。



## ResourceLoader

ResourceLoader接口用于加载各种路径下的资源，比如类路径下资源，文件系统中的资源等。ResourceLoader的默认实现DefaultResourceLoader，它的Resource getResource(String location) 方法会通过先注册的ProtocolResolver去尝试解析location，如果某个ProtocolResolver解析成功，直接返回一个Resource。如果解析失败，会依次判断是否以/开头，或者前缀为"classpath:"，如果都不满足则会创建以个URL，根据URL的协议读取本地文件或远程资源，具体代码如下：
```Java
public Resource getResource(String location) {
	Assert.notNull(location, "Location must not be null");

	for (ProtocolResolver protocolResolver : this.protocolResolvers) {
		Resource resource = protocolResolver.resolve(location, this);
		if (resource != null) {
			return resource;
		}
	}

	if (location.startsWith("/")) {
		return getResourceByPath(location);
	}
	else if (location.startsWith(CLASSPATH_URL_PREFIX)) {
		return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
	}
	else {
		try {
			// Try to parse the location as a URL...
			URL url = new URL(location);
			return (ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
		}
		catch (MalformedURLException ex) {
			// No URL -> resolve as resource path.
			return getResourceByPath(location);
		}
	}
}
```


## ApplicationEventPublisher

ApplicationEventPublisher接口用于发布ApplicationEvent事件。最终事件会通知到各个ApplicationListener。

{% asset_img ApplicationEvent.png ApplicationEvent类图 %}


## MessageSource

MessageSource主要用于解析消息，支持参数化和国际化。


# ApplicationContext 继承关系

{% asset_img ApplicationContext.png ApplicationContext类图 %}

ApplicationContext接口有两个子接口分支：ConfigurableApplicationContext和WebApplicationContext。ConfigurableApplicationContext添加了配置功能，继承了Lifecycle和Closeable，提供了生命周期管理：addApplicationListener为ApplicationEventPublisher接口提供了ApplicationListener,addProtocolResolver方法提供了添加Resource解析器的功能，setEnvironment为EnvironmentCapable接口提供了修改Environment的功能，setId提供了修改ApplicationContext Id的功能，setParent为HierarchicalBeanFactory提供了设置父ApplicationContext和父BeanFactory的功能， 此外 ConfigurableApplicationContext还提供了refresh用于刷新，isActive判断状态，close方法用于关闭和释放资源，并且addBeanFactoryPostProcessor提供了添加BeanFactoryPostProcessor的功能，这些BeanFactoryPostProcessor将会在ApplicationContext内部的BeanFactory refresh时，评估任何BeanDefinition之前被调用。而WebApplicationContext提供了获取ServletContext方法，以及SCOPE_REQUEST、SCOPE_SESSION和SCOPE_APPLICATION Bean。


ConfigurableApplicationContext有一个继承接口ConfigurableWebApplicationContext和抽象子类AbstractApplicationContext。



AbstractApplicationContext类通过模板方法设计模式为子类提供了框架，具体的实现在子类中。其中最重要的是refresh方法。

```Java
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
		// 1. 为刷新context做准备，加载属性文件，设置环境信息和context状态信息
		prepareRefresh();

		// Tell the subclass to refresh the internal bean factory.
		// 2. 获取内部BeanFactory,其中调用了refreshBeanFactory和getBeanFactory，由子类实现
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

		// 3. 配置beanFactory的标准上下特征，比如ClassLoader和post-processors.
		prepareBeanFactory(beanFactory);

		try {
			// 4. application context的内部bean Factory初始化完成后修改它，所有的bean definition都已经加载了，但还没有实例化，这个方法可以用来注册特殊的BeanPostProcessors
			postProcessBeanFactory(beanFactory);

			// Invoke factory processors registered as beans in the context.
			// 5. 实例化所有的BeanFactoryPostProcessor bean，并按照指定的顺序调用，在singleton 实例化之前调用
			invokeBeanFactoryPostProcessors(beanFactory);

			// 6. 注册拦截Bean创建的BeanPostProcessor
			registerBeanPostProcessors(beanFactory);

			// 7. 初始化Message资源
			initMessageSource();

			// 8. 初始化事件广播器
			initApplicationEventMulticaster();

			// 9. 留给子类初始化其他特殊的Bean
			onRefresh();

			// 10. 在所有注册Bean中查找ApplicationListener Bean,并注册到消息广播器中
			registerListeners();

			// 11. 初始化剩下的单例Bean(非延迟加载的)
			finishBeanFactoryInitialization(beanFactory);

			// 12. 完成刷新过程,通知生命周期处理器lifecycleProcessor刷新过程,同时发出ContextRefreshEvent
			finishRefresh();
		}

		catch (BeansException ex) {
			if (logger.isWarnEnabled()) {
				logger.warn("Exception encountered during context initialization - " +
						"cancelling refresh attempt: " + ex);
			}

			// Destroy already created singletons to avoid dangling resources.
			destroyBeans();

			// Reset 'active' flag.
			cancelRefresh(ex);

			// Propagate exception to caller.
			throw ex;
		}

		finally {
			// Reset common introspection caches in Spring's core, since we
			// might not ever need metadata for singleton beans anymore...
			resetCommonCaches();
		}
	}
}
```


obtainFreshBeanFactory方法中调用了refreshBeanFactory和getBeanFactory方法，refreshBeanFactory和getBeanFactory方法都是模板方法，由子类去实现。AbstractApplicationContext的两个子类：GenericApplicationContext和AbstractRefreshableApplicationContext都提供了refreshBeanFactory的实现，GenericApplicationContext的实现只是设置refreshed和serializationId，AbstractRefreshableApplicationContext则会去创建DefaultListableBeanFactory，并调用loadBeanDefinitions加载BeanDefinition,loadBeanDefinitions方法也是一个抽象方法，具体实现由AbstractRefreshableApplicationContext子类实现。


ConfigurableWebApplicationContext是所有webContext的父接口，提供了提供了设置web信息的方法：

```Java
void setServletContext(@Nullable ServletContext servletContext);
void setNamespace(@Nullable String namespace);
void setConfigLocation(String configLocation);
void setConfigLocations(String... configLocations);

```




