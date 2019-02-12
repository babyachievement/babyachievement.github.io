---
title: Spring-8-AOP设计与实现
date: 2019-01-10 11:53:35
tags: ["Spring", "AOP"]
---
 
　　　　Spring AOP核心技术是代理，为了让Spring AOP起作用，Spring内部需要完成一系列过程，如使用JDK动态代理或者CGLIB创建代理对象，然后启动代理对象拦截器完成横切面的织入。

<!-- more -->

## 代理对象的创建

　　　　Spring中通过配置和调用ProxyFactoryBean来生成代理对象，在生成过程中可以使用JDK的Proxy和CGLIB两种生成方式。

{% asset_img ProxyFactoryBean.png ProxyFactoryBean类图 %}

　　　　由上图可以看出，完成AOP应用的类有AspectJProxyFactory、ProxyFactory和ProxyFactoryBean，它们都是ProxyConfig、AdvisedSupport和ProxyCreatorSupport的子类。ProxyConfig为子类提供类配置属性；AdvisedSupport封装了对通着和通知器的相关操作，这些操作对于不同AOP代理对象的生成都是一样的，但对于具体的AOP代理对象的创建，AdvisedSupport则将其委托给它的子类去完成；ProxyCreatorSupport可以看做子类创建AOP代理对象的辅助类。具体AOP代理对象的生成，根据需要分别由ProxyFactoryBea、AspectJProxyFactory和ProxyFactory完成。对于使用AspectJ的AOP应用，AspectJProxyFactory起到集成Spring和AspectJ的作用；对于使用Spring AOP的应用，ProxyFactoryBean和ProxyFactory都提供了AOP功能的封装，只是使用ProxyFactoryBean可以做IoC容器中完成声明式配置，而使用ProxyFactory，则需要编程式地使用Spring AOP功能。

### ProxyFactory

{% asset_img ProxyFactory.png ProxyFactory调用栈 %}

　　　　AspectJAwareAdvisorAutoProxyCreator实现了BeanPostProcessor接口，在Bean被创建后AspectJAwareAdvisorAutoProxyCreator的postProcessAfterInitialization会被调用，postProcessAfterInitialization是在AspectJAwareAdvisorAutoProxyCreator的父类AbstractAutoProxyCreator中定义的，该方法会调用wrapIfNecessary查找满足条件的Advisor，然后为其常见代理对象。代理对象的创建会委托给AopProxyFactory，Spring提供了默认的AOPProxyFactory：DefaultAopProxyFactory，DefaultAopProxyFactory根据判断结果为其创建Java Proxy或Cglib Proxy。
```Java
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
	if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
		Class<?> targetClass = config.getTargetClass();
		if (targetClass == null) {
			throw new AopConfigException("TargetSource cannot determine target class: " +
					"Either an interface or a target is required for proxy creation.");
		}
		if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
			return new JdkDynamicAopProxy(config);
		}
		return new ObjenesisCglibAopProxy(config);
	}
	else {
		return new JdkDynamicAopProxy(config);
	}
}
```

　　　　由上面的代码可以看到，Spring并没有直接创建一个CglibAopProxy，而是创建了一个ObjenesisCglibAopProxy，通过使用Objenesis，不需要代理类有默认构造函数，并且如果使用Objenesis创建代理对象失败，其会自动fall back使用Cglib创建代理对象，具体参考Spring git commit:1f9e8f68d445ae1d16e27ccde97403efaaa23a81

```
@SuppressWarnings("unchecked")
protected Object createProxyClassAndInstance(Enhancer enhancer, Callback[] callbacks) {
	Class<?> proxyClass = enhancer.createClass();
	Object proxyInstance = null;

	if (objenesis.isWorthTrying()) {
		try {
			proxyInstance = objenesis.newInstance(proxyClass, enhancer.getUseCache());
		}
		catch (Throwable ex) {
			logger.debug("Unable to instantiate proxy using Objenesis, " +
					"falling back to regular proxy construction", ex);
		}
	}

	if (proxyInstance == null) {
		// Regular instantiation via default constructor...
		try {
			Constructor<?> ctor = (this.constructorArgs != null ?
					proxyClass.getDeclaredConstructor(this.constructorArgTypes) :
					proxyClass.getDeclaredConstructor());
			ReflectionUtils.makeAccessible(ctor);
			proxyInstance = (this.constructorArgs != null ?
					ctor.newInstance(this.constructorArgs) : ctor.newInstance());
		}
		catch (Throwable ex) {
			throw new AopConfigException("Unable to instantiate proxy using Objenesis, " +
					"and regular proxy instantiation via default constructor fails as well", ex);
		}
	}

	((Factory) proxyInstance).setCallbacks(callbacks);
	return proxyInstance;
}
```

[Add ability to create proxy around classes that has no default constructor](https://jira.spring.io/browse/SPR-10594?redirect=false)
[SPRING 4: CGLIB-BASED PROXY CLASSES WITH NO DEFAULT CONSTRUCTOR](https://blog.codeleak.pl/2014/07/spring-4-cglib-based-proxy-classes-with-no-default-ctor.html)



