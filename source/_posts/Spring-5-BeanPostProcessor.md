---
title: Spring-1-BeanPostProcessor
date: 2018-12-25 10:06:28
tags: ["Spring"]
---

BeanPostProcessor是Spring容器的一个扩展点，可以进行自定义的实例化、初始化、依赖装配、依赖检查等流程，即可以覆盖默认的实例化，也可以增强初始化、依赖注入、依赖检查等流程。BeanPostProcessor一共有两个回调方法postProcessBeforeInitialization和postProcessAfterInitialization，那这两个方法会在什么Spring执行流程中的哪个步骤执行呢？

<!-- more -->

```Java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {

	if (logger.isTraceEnabled()) {
		logger.trace("Creating instance of bean '" + beanName + "'");
	}
	RootBeanDefinition mbdToUse = mbd;

	// Make sure bean class is actually resolved at this point, and
	// clone the bean definition in case of a dynamically resolved Class
	// which cannot be stored in the shared merged bean definition.
	Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
	if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
		mbdToUse = new RootBeanDefinition(mbd);
		mbdToUse.setBeanClass(resolvedClass);
	}

	// Prepare method overrides.
	try {
		mbdToUse.prepareMethodOverrides();
	}
	catch (BeanDefinitionValidationException ex) {
		throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
				beanName, "Validation of method overrides failed", ex);
	}

	try {
		// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
		Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
		if (bean != null) {
			return bean;
		}
	}
	catch (Throwable ex) {
		throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
				"BeanPostProcessor before instantiation of bean failed", ex);
	}

	try {
		Object beanInstance = doCreateBean(beanName, mbdToUse, args);
		return beanInstance;
	}
	catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
		// A previously detected exception with proper bean creation context already,
		// or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
		throw ex;
	}
	catch (Throwable ex) {
		throw new BeanCreationException(
				mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
	}
}
```


BeanPostProcessor在Bean创建过程中的扩展点：

protected Object createBean(final String beanName, final RootBeanDefinition mbd, final Object[] args); 创建Bean

（1、resolveBeanClass(mbd, beanName); 解析Bean class，若class配置错误将抛出CannotLoadBeanClassException；

 

（2、mbd.prepareMethodOverrides(); 准备和验证配置的方法注入，若验证失败抛出BeanDefinitionValidationException

有关方法注入知识请参考【第三章】 DI 之 3.3 更多DI的知识 ——跟我学spring3 3.3.5 方法注入；

 

（3、Object bean = resolveBeforeInstantiation(beanName, mbd); 第一个BeanPostProcessor扩展点，此处只执行InstantiationAwareBeanPostProcessor类型的BeanPostProcessor Bean；

（3.1、bean = applyBeanPostProcessorsBeforeInstantiation(mbd.getBeanClass(), beanName);执行InstantiationAwareBeanPostProcessor的实例化的预处理回调方法postProcessBeforeInstantiation（自定义的实例化，如创建代理）；

（3.2、bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);执行InstantiationAwareBeanPostProcessor的实例化的后处理回调方法postProcessAfterInitialization（如依赖注入），如果3.1处返回的Bean不为null才执行；

 

（4、如果3处的扩展点返回的bean不为空，直接返回该bean，后续流程不需要执行；

 

（5、Object beanInstance = doCreateBean(beanName, mbd, args); 执行spring的创建bean实例的流程；

 

 

（6、createBeanInstance(beanName, mbd, args); 实例化Bean

（6.1、instantiateUsingFactoryMethod 工厂方法实例化；请参考【http://jinnianshilongnian.iteye.com/blog/1413857】

（6.2、构造器实例化，请参考【http://jinnianshilongnian.iteye.com/blog/1413857】；

（6.2.1、如果之前已经解析过构造器

（6.2.1.1 autowireConstructor：有参调用autowireConstructor实例化

（6.2.1.2、instantiateBean：无参调用instantiateBean实例化；

（6.2.2、如果之前没有解析过构造器：

 

（6.2.2.1、通过SmartInstantiationAwareBeanPostProcessor的determineCandidateConstructors回调方法解析构造器，第二个BeanPostProcessor扩展点，返回第一个解析成功（返回值不为null）的构造器组，如AutowiredAnnotationBeanPostProcessor实现将自动扫描通过@Autowired/@Value注解的构造器从而可以完成构造器注入，请参考【第十二章】零配置 之 12.2 注解实现Bean依赖注入 ——跟我学spring3 ；

（6.2.2.2、autowireConstructor：如果（6.2.2.1返回的不为null，且是有参构造器，调用autowireConstructor实例化；

（6.2.2.3、instantiateBean： 否则调用无参构造器实例化；

 

（7、applyMergedBeanDefinitionPostProcessors(mbd, beanType,beanName);第三个BeanPostProcessor扩展点，执行Bean定义的合并；

（7.1、执行MergedBeanDefinitionPostProcessor的postProcessMergedBeanDefinition回调方法，进行bean定义的合并；

 

（8、addSingletonFactory(beanName, new ObjectFactory() {

                            public Object getObject() throws BeansException {

                                   return getEarlyBeanReference(beanName, mbd, bean);

                            }

                     });  及早暴露单例Bean引用，从而允许setter注入方式的循环引用

（8.1、SmartInstantiationAwareBeanPostProcessor的getEarlyBeanReference；第四个BeanPostProcessor扩展点，当存在循环依赖时，通过该回调方法获取及早暴露的Bean实例；

 

（9、populateBean(beanName, mbd, instanceWrapper);装配Bean依赖

（9.1、InstantiationAwareBeanPostProcessor的postProcessAfterInstantiation；第五个BeanPostProcessor扩展点，在实例化Bean之后，所有其他装配逻辑之前执行，如果false将阻止其他的InstantiationAwareBeanPostProcessor的postProcessAfterInstantiation的执行和从（9.2到（9.5的执行，通常返回true；

（9.2、autowireByName、autowireByType：根据名字和类型进行自动装配，自动装配的知识请参考【第三章】 DI 之 3.3 更多DI的知识 ——跟我学spring3  3.3.3  自动装配；

（9.3、InstantiationAwareBeanPostProcessor的postProcessPropertyValues：第六个BeanPostProcessor扩展点，完成其他定制的一些依赖注入，如AutowiredAnnotationBeanPostProcessor执行@Autowired注解注入，CommonAnnotationBeanPostProcessor执行@Resource等注解的注入，PersistenceAnnotationBeanPostProcessor执行@ PersistenceContext等JPA注解的注入，RequiredAnnotationBeanPostProcessor执行@ Required注解的检查等等，请参考【第十二章】零配置 之 12.2 注解实现Bean依赖注入 ——跟我学spring3；

（9.4、checkDependencies：依赖检查，请参考【第三章】 DI 之 3.3 更多DI的知识 ——跟我学spring3  3.3.4  依赖检查；

（9.5、applyPropertyValues：应用明确的setter属性注入，请参考【第三章】 DI 之 3.1 DI的配置使用 ——跟我学spring3 ；

 

（10、exposedObject = initializeBean(beanName, exposedObject, mbd); 执行初始化Bean流程；

（10.1、invokeAwareMethods（BeanNameAware、BeanClassLoaderAware、BeanFactoryAware）：调用一些Aware标识接口注入如BeanName、BeanFactory；

（10.2、BeanPostProcessor的postProcessBeforeInitialization：第七个扩展点，在调用初始化之前完成一些定制的初始化任务，如BeanValidationPostProcessor完成JSR-303 @Valid注解Bean验证，InitDestroyAnnotationBeanPostProcessor完成@PostConstruct注解的初始化方法调用，ApplicationContextAwareProcessor完成一些Aware接口的注入（如EnvironmentAware、ResourceLoaderAware、ApplicationContextAware），其返回值将替代原始的Bean对象；

（10.3、invokeInitMethods ： 调用初始化方法；

（10.3.1、InitializingBean的afterPropertiesSet ：调用InitializingBean的afterPropertiesSet回调方法；

（10.3.2、通过xml指定的自定义init-method ：调用通过xml配置的自定义init-method

（10.3.3、BeanPostProcessor的postProcessAfterInitialization ：第八个扩展点，AspectJAwareAdvisorAutoProxyCreator（完成xml风格的AOP配置(<aop:config>)的目标对象包装到AOP代理对象）、AnnotationAwareAspectJAutoProxyCreator（完成@Aspectj注解风格（<aop:aspectj-autoproxy> @Aspect）的AOP配置的目标对象包装到AOP代理对象），其返回值将替代原始的Bean对象；

 

（11、if (earlySingletonExposure) {

                     Object earlySingletonReference = getSingleton(beanName, false);

            ……

      } ：如果是earlySingleExposure，调用getSingle方法获取Bean实例；

earlySingleExposure =(mbd.isSingleton() && this.allowCircularReferences && isSingletonCurrentlyInCreation(beanName))

只要单例Bean且允许循环引用（默认true）且当前单例Bean正在创建中

（11.1、如果是earlySingletonExposure调用getSingleton将触发【8】处ObjectFactory.getObject()的调用，通过【8.1】处的getEarlyBeanReference获取相关Bean（如包装目标对象的代理Bean）；（在循环引用Bean时可能引起Spring事务处理时自我调用的解决方案及一些实现方式的风险）；

 

（12、registerDisposableBeanIfNecessary(beanName, bean, mbd) ： 注册Bean的销毁方法（只有非原型Bean可注册）；

（12.1、单例Bean的销毁流程

（12.1.1、DestructionAwareBeanPostProcessor的postProcessBeforeDestruction ： 第九个扩展点，如InitDestroyAnnotationBeanPostProcessor完成@PreDestroy注解的销毁方法注册和调用；

（12.1.2、DisposableBean的destroy：注册/调用DisposableBean的destroy销毁方法；

（12.1.3、通过xml指定的自定义destroy-method ： 注册/调用通过XML指定的destroy-method销毁方法；

（12.1.2、Scope的registerDestructionCallback：注册自定义的Scope的销毁回调方法，如RequestScope、SessionScope等；其流程和【12.1 单例Bean的销毁流程一样】，关于自定义Scope请参考【第三章】 DI 之 3.4 Bean的作用域 ——跟我学spring3

 

（13、到此Bean实例化、依赖注入、初始化完毕可以返回创建好的bean了。


InstantiationAwareBeanPostProcessor

MergedBeanDefinitionPostProcessor 